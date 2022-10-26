---
title: collection.query.stream()
description: Process query results as a stream
---

Process query results as a stream.

```javascript
import { collection } from '@nitric/sdk';

const profiles = collection('profiles');

const profileQuery = profiles.query();

const results = await profileQuery.stream();
```

## Examples

### Streaming results from a query

```javascript
import { collection } from '@nitric/sdk';

const profiles = collection('profiles');

const profileQuery = profiles.query();

let results = await profileQuery.stream();

results.on('data', (doc) => {
  // handle stream results
});

results.on('end', () => {
  // handle stream closing
});
```

## See also

- [query().where()](./collection-query-where)
- [query().limit()](./collection-query-limit)
- [query().pagingFrom()](./collection-query-pagingfrom)