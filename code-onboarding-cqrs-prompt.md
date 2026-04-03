# Code QA Task — Code Onboarding & Comprehension (TypeScript)

## prompt_statement

I just joined this team last week and I'm supposed to add a `CancelOrderCommand` to the orders service, but before I touch anything I need to understand what's actually happening in this messaging module. I've read through all the files below and I have three specific things I can't wrap my head around: (1) why commands and queries live on completely separate buses instead of one unified bus, (2) what the `reduceRight` call in `dispatch()` is doing and why the pipeline is built that way rather than just looping through middleware in order, and (3) why `CreateOrderValidator` registers itself into a global map as a side effect of importing the file — that feels wrong to me but maybe there's a reason. Can you walk me through the full execution flow from the moment `commandBus.dispatch(new CreateOrderCommand(...))` is called in `bootstrap.ts` all the way through to the handler returning, touching every layer that gets involved?

---

```
// FILE: src/messaging/types.ts

export abstract class Command {
  abstract readonly commandType: string;
  readonly occurredAt: Date = new Date();
}

export abstract class Query<TResult> {
  abstract readonly queryType: string;
  readonly __resultType?: TResult; // phantom type anchor — never assigned at runtime
}

export type ExtractQueryResult<Q> = Q extends Query<infer R> ? R : never;

export interface CommandHandler<TCommand extends Command = Command> {
  commandType: string;
  handle(command: TCommand): Promise<void>;
}

export interface QueryHandler<TQuery extends Query<unknown> = Query<unknown>> {
  queryType: string;
  handle(query: TQuery): Promise<ExtractQueryResult<TQuery>>;
}

export type Next = () => Promise<void>;
export type MiddlewareFn = (command: Command, next: Next) => Promise<void>;

export class HandlerNotFoundError extends Error {
  constructor(type: string) {
    super(`No handler registered for type: "${type}"`);
    this.name = 'HandlerNotFoundError';
  }
}

export class HandlerAlreadyRegisteredError extends Error {
  constructor(type: string) {
    super(`A handler is already registered for type: "${type}"`);
    this.name = 'HandlerAlreadyRegisteredError';
  }
}
```

---

```
// FILE: src/messaging/command-bus.ts

import {
  Command,
  CommandHandler,
  HandlerNotFoundError,
  HandlerAlreadyRegisteredError,
  MiddlewareFn,
  Next,
} from './types';

export class CommandBus {
  private handlers = new Map<string, CommandHandler>();
  private middlewareStack: MiddlewareFn[] = [];

  register(handler: CommandHandler): void {
    if (this.handlers.has(handler.commandType)) {
      throw new HandlerAlreadyRegisteredError(handler.commandType);
    }
    this.handlers.set(handler.commandType, handler);
  }

  use(middleware: MiddlewareFn): void {
    this.middlewareStack.push(middleware);
  }

  async dispatch(command: Command): Promise<void> {
    const handler = this.handlers.get(command.commandType);
    if (!handler) {
      throw new HandlerNotFoundError(command.commandType);
    }

    const leaf: Next = () => handler.handle(command);

    const pipeline = this.middlewareStack.reduceRight<Next>(
      (next, middleware) => () => middleware(command, next),
      leaf,
    );

    await pipeline();
  }
}
```

---

```
// FILE: src/messaging/query-bus.ts

import {
  Query,
  QueryHandler,
  HandlerNotFoundError,
  HandlerAlreadyRegisteredError,
  ExtractQueryResult,
} from './types';

export class QueryBus {
  private handlers = new Map<string, QueryHandler>();

  register(handler: QueryHandler): void {
    if (this.handlers.has(handler.queryType)) {
      throw new HandlerAlreadyRegisteredError(handler.queryType);
    }
    this.handlers.set(handler.queryType, handler);
  }

  async dispatch<TQuery extends Query<unknown>>(
    query: TQuery,
  ): Promise<ExtractQueryResult<TQuery>> {
    const handler = this.handlers.get((query as Query<unknown> & { queryType: string }).queryType);
    if (!handler) {
      throw new HandlerNotFoundError((query as Query<unknown> & { queryType: string }).queryType);
    }
    return handler.handle(query) as Promise<ExtractQueryResult<TQuery>>;
  }
}
```

---

```
// FILE: src/messaging/middleware/logging.middleware.ts

import { MiddlewareFn, Command } from '../types';

export interface Logger {
  info(message: string, meta?: Record<string, unknown>): void;
  error(message: string, meta?: Record<string, unknown>): void;
}

export function createLoggingMiddleware(logger: Logger): MiddlewareFn {
  return async (command: Command, next) => {
    const start = Date.now();
    const commandType = command.commandType;

    try {
      await next();
      const durationMs = Date.now() - start;
      logger.info('Command succeeded', { commandType, durationMs });
    } catch (err) {
      const durationMs = Date.now() - start;
      logger.error('Command failed', {
        commandType,
        durationMs,
        error: err instanceof Error ? err.message : String(err),
      });
      throw err;
    }
  };
}
```

---

```
// FILE: src/messaging/middleware/validation.middleware.ts

import { MiddlewareFn, Command } from '../types';

export interface ValidationResult {
  valid: boolean;
  errors: string[];
}

export interface Validator<T> {
  commandType: string;
  validate(command: T): ValidationResult;
}

export class ValidationError extends Error {
  constructor(public readonly errors: string[]) {
    super(`Validation failed: ${errors.join('; ')}`);
    this.name = 'ValidationError';
  }
}

// Module-level registry: populated by side-effecting imports in feature modules
export const validatorRegistry = new Map<string, Validator<Command>>();

export function registerValidator<T extends Command>(validator: Validator<T>): void {
  validatorRegistry.set(validator.commandType, validator as Validator<Command>);
}

export const commandValidationMiddleware: MiddlewareFn = async (command, next) => {
  const validator = validatorRegistry.get(command.commandType);
  if (validator) {
    const result = validator.validate(command);
    if (!result.valid) {
      throw new ValidationError(result.errors);
    }
  }
  await next();
};
```

---

```
// FILE: src/orders/commands/create-order.ts

import { Command, CommandHandler } from '../../messaging/types';
import {
  Validator,
  ValidationResult,
  registerValidator,
} from '../../messaging/middleware/validation.middleware';

// ── Domain types ──────────────────────────────────────────────────────────────

export interface OrderItem {
  productId: string;
  quantity: number;
  unitPriceCents: number;
}

// ── Command ───────────────────────────────────────────────────────────────────

export class CreateOrderCommand extends Command {
  readonly commandType = 'orders.create';

  constructor(
    public readonly customerId: string,
    public readonly items: OrderItem[],
  ) {
    super();
  }
}

// ── Validator (registers itself on module load) ───────────────────────────────

class CreateOrderValidator implements Validator<CreateOrderCommand> {
  readonly commandType = 'orders.create';

  validate(command: CreateOrderCommand): ValidationResult {
    const errors: string[] = [];

    if (!command.customerId || command.customerId.trim() === '') {
      errors.push('customerId is required');
    }
    if (!command.items || command.items.length === 0) {
      errors.push('Order must contain at least one item');
    } else {
      command.items.forEach((item, i) => {
        if (item.quantity <= 0) {
          errors.push(`Item[${i}].quantity must be greater than zero`);
        }
        if (item.unitPriceCents < 0) {
          errors.push(`Item[${i}].unitPriceCents must be non-negative`);
        }
      });
    }

    return { valid: errors.length === 0, errors };
  }
}

// Side-effecting registration: runs once when this module is first imported
registerValidator(new CreateOrderValidator());

// ── Repository interface ──────────────────────────────────────────────────────

export interface OrderRepository {
  save(order: {
    id: string;
    customerId: string;
    items: OrderItem[];
    createdAt: Date;
    status: 'pending';
  }): Promise<void>;
}

// ── Handler ───────────────────────────────────────────────────────────────────

export class CreateOrderHandler implements CommandHandler<CreateOrderCommand> {
  readonly commandType = 'orders.create';

  constructor(private readonly repository: OrderRepository) {}

  async handle(command: CreateOrderCommand): Promise<void> {
    const id = `ord_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
    await this.repository.save({
      id,
      customerId: command.customerId,
      items: command.items,
      createdAt: command.occurredAt,
      status: 'pending',
    });
  }
}
```

---

```
// FILE: src/orders/queries/get-order.ts

import { Query, QueryHandler } from '../../messaging/types';

// ── DTO ───────────────────────────────────────────────────────────────────────

export interface OrderDto {
  id: string;
  customerId: string;
  status: string;
  totalCents: number;
  createdAt: string; // ISO-8601
}

// ── Query ─────────────────────────────────────────────────────────────────────

export class GetOrderQuery extends Query<OrderDto> {
  readonly queryType = 'orders.get';

  constructor(
    public readonly orderId: string,
    public readonly requestingCustomerId: string,
  ) {
    super();
  }
}

// ── Repository interface ──────────────────────────────────────────────────────

export interface OrderReadRepository {
  findById(id: string): Promise<OrderDto | null>;
}

// ── Handler ───────────────────────────────────────────────────────────────────

export class GetOrderHandler implements QueryHandler<GetOrderQuery> {
  readonly queryType = 'orders.get';

  constructor(private readonly readRepository: OrderReadRepository) {}

  async handle(query: GetOrderQuery): Promise<OrderDto> {
    const order = await this.readRepository.findById(query.orderId);

    if (!order) {
      throw new Error(`Order not found: ${query.orderId}`);
    }

    if (order.customerId !== query.requestingCustomerId) {
      throw new Error(`Access denied: order does not belong to customer ${query.requestingCustomerId}`);
    }

    return order;
  }
}
```

---

```
// FILE: src/messaging/index.ts

// Core types
export {
  Command,
  Query,
  ExtractQueryResult,
  CommandHandler,
  QueryHandler,
  MiddlewareFn,
  Next,
  HandlerNotFoundError,
  HandlerAlreadyRegisteredError,
} from './types';

// Buses
export { CommandBus } from './command-bus';
export { QueryBus } from './query-bus';

// Middleware
export { createLoggingMiddleware } from './middleware/logging.middleware';
export type { Logger } from './middleware/logging.middleware';
export {
  commandValidationMiddleware,
  registerValidator,
  validatorRegistry,
  ValidationError,
} from './middleware/validation.middleware';
export type { Validator, ValidationResult } from './middleware/validation.middleware';
```

---

```
// FILE: src/app/bootstrap.ts

// Importing create-order triggers the side-effecting validator registration
// before the bus is used — this must come before commandBus.dispatch() is called
import { CreateOrderCommand, CreateOrderHandler } from '../orders/commands/create-order';
import { GetOrderQuery, GetOrderHandler, OrderDto } from '../orders/queries/get-order';
import {
  CommandBus,
  QueryBus,
  createLoggingMiddleware,
  commandValidationMiddleware,
  Logger,
} from '../messaging';

// ── Stub logger ───────────────────────────────────────────────────────────────

const consoleLogger: Logger = {
  info: (message, meta) => console.log(`[INFO] ${message}`, meta ?? {}),
  error: (message, meta) => console.error(`[ERROR] ${message}`, meta ?? {}),
};

// ── Wire up CommandBus ────────────────────────────────────────────────────────

const commandBus = new CommandBus();

commandBus.use(createLoggingMiddleware(consoleLogger));
commandBus.use(commandValidationMiddleware);

// Stub write repository
const stubOrderRepository = {
  async save(order: Parameters<typeof stubOrderRepository.save>[0]): Promise<void> {
    console.log('[STUB] Saved order:', order);
  },
};

commandBus.register(new CreateOrderHandler(stubOrderRepository));

// ── Wire up QueryBus ──────────────────────────────────────────────────────────

const queryBus = new QueryBus();

// Stub read repository (returns a hardcoded order for demonstration)
const stubOrderReadRepository = {
  async findById(id: string): Promise<OrderDto | null> {
    if (id === 'ord_demo') {
      return {
        id: 'ord_demo',
        customerId: 'cust_abc',
        status: 'pending',
        totalCents: 4998,
        createdAt: new Date().toISOString(),
      };
    }
    return null;
  },
};

queryBus.register(new GetOrderHandler(stubOrderReadRepository));

// ── Demo execution ────────────────────────────────────────────────────────────

async function main(): Promise<void> {
  // Dispatching a command — triggers logging → validation → handler
  await commandBus.dispatch(
    new CreateOrderCommand('cust_abc', [
      { productId: 'prod_1', quantity: 2, unitPriceCents: 1999 },
      { productId: 'prod_2', quantity: 1, unitPriceCents: 1000 },
    ]),
  );

  // Dispatching a query — no middleware, returns typed OrderDto
  const order = await queryBus.dispatch(new GetOrderQuery('ord_demo', 'cust_abc'));
  console.log('[QUERY RESULT]', order);
}

main().catch(console.error);
```
