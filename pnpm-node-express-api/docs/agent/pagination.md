## 8. Pagination — Offset & Cursor

> Read this when adding pagination (offset or cursor) to list endpoints.

### Offset Pagination (default for admin / small tables)

Utility `src/utils/pagination.js` (ESM):
```js
export const buildPagination = (total, limit, offset) => {
  const totalPages = Math.ceil(total / limit);
  const currentPage = Math.floor(offset / limit) + 1;
  return { total, limit, offset, currentPage, totalPages, hasNextPage: offset + limit < total, hasPrevPage: offset > 0 };
};
```
- Controller reads `req.query.limit` / `req.query.offset` (or `req.body` for POST search).
- Service normalizes and delegates to `repository.count()` + `repository.findAll({ limit, offset, order, transaction })`.
- Do not trust client values; always clamp. Enforce `limit` max via Joi (`max 500`).

### Cursor Pagination (preferred for large / infinite-scroll lists)

- Use when table >10k rows or frontend uses infinite scroll. Avoids slow `OFFSET` and stable under inserts.
- **Contract:** `GET /api/v1/users?limit=20&cursor=42` where `cursor` is the last seen PK (`idUser`). First page: no `cursor`.
- **Repository:** `where: cursor ? { idUser: { [Op.gt]: cursor } } : {}`, `order:[['idUser','ASC']]`, `limit: limit+1` (extra row to know `hasNextPage`).
- **Helper** (`src/utils/pagination.js`):
  ```js
  export const buildCursorPagination = (rows, limit) => {
    const hasNextPage = rows.length > limit;
    const data = hasNextPage ? rows.slice(0, -1) : rows;
    const nextCursor = hasNextPage ? data[data.length-1].idUser : null;
    return { data, pagination: { limit, nextCursor, hasNextPage } };
  };
  ```
- **Schema:** `cursor: Joi.number().integer().positive().allow(null)`, `limit: Joi.number().integer().min(1).max(100)`.
- **Rule:** offset and cursor are mutually exclusive — validate `Joi.object({ limit, offset, cursor }).xor('offset','cursor')` if you support both; document which endpoint uses which.

---
