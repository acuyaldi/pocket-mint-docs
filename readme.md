# Pocket Mint Documentation

Shared product and architecture documentation for Pocket Mint.

## Repositories

- `pocket-mint-fe` — Next.js frontend
- `pocket-mint-be` — backend API
- `pocket-mint-docs` — shared product and architecture documentation

## Product Documentation

- [Product RFC](docs/product/rfc.md)
- [Design System](docs/product/design-system.md)

## Planned Documentation

- Screen specification
- Component specification
- User flows
- System architecture
- Implementation reconciliation report

## Source-of-truth rules

- Product behavior and financial rules: `docs/product/rfc.md`
- Database implementation: backend Prisma schema
- API implementation: backend routes and services
- UI implementation: frontend source code
- Visual direction: `docs/product/design-system.md`

When documents and implementation disagree, record the discrepancy before changing either side.