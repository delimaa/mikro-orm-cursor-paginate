## Description

Cursor pagination support for MikroORM following [Relay GraphQL Cursor Connections Specification](https://relay.dev/graphql/connections.htm).


*NOTE: Since v6, MikroORM natively supports cursor based pagination using `em.findByCursor()`. As such, this package is not necessary for MikroORM v6. It is only relevant for projects in MikroORM v5.*

## Installation

```bash
npm install @delimaa/mikro-orm-cursor-pagination
```

## Usage

```ts
import { cursorPaginationFind } from '@delimaa/mikro-orm-cursor-pagination';

@Entity()
class User {
  @PrimaryKey()
  id!: number;

  @Property()
  name!: string;

  @Property()
  age!: number;

  @ManyToMany()
  friends = new Collection<User>(this);
}

// Initial cursor pagination when no cursor is provided
let page = await cursorPaginationFind(
  em,
  User,
  // 👇 Initial pagination when no cursor is provided, pass `first` & `orderBy`
  {
    first: 10,
    orderBy: {
      name: 'ASC',
      age: 'DESC',
      id: 'ASC',
    },
  },
  // 👇 Pass any native MikroORM where condition
  {
    age: { $gt: 20 },
  },
  // 👇 Pass any native MikroORM options except `limit`, `offset` and `orderBy` properties which are not allowed
  {
    populate: ['friends'],
  },
);

page.totalCount; // Total count of records
page.pageInfo.hasNextPage; // Whether next page is available
page.pageInfo.hasPreviousPage; // Whether previous page is available
page.pageInfo.startCursor; // Start cursor
page.pageInfo.endCursor; // End cursor
page.edges; // Array of edges containing cursor and node { cursor: string, node: Loaded<User, 'friends'> }

// Forward cursor pagination
page = await cursorPaginationFind(
  em,
  User,
  // 👇 For forward cursor pagination, pass `first` & `after`
  {
    first: 10,
    after: page.pageInfo.endCursor,
  },
  {
    age: { $gt: 20 },
  },
  {
    populate: ['friends'],
  },
);

// Backward cursor pagination
page = await cursorPaginationFind(
  em,
  User,
  // 👇 For backward cursor pagination, pass `last` & `before`
  {
    last: 10,
    before: page.pageInfo.startCursor,
  },
  {
    age: { $gt: 20 },
  },
  {
    populate: ['friends'],
  },
);

```
