# SQLs de Verificação — Sprint 2 (TASK-01 + TASK-04)

> Migration: `20260414130000_add_approval_workflow`
> Executar contra o banco `sih` após aplicar a migration.

---

## 1. Verificar que a migration foi aplicada

```sql
SELECT migration_name, finished_at
FROM _prisma_migrations
WHERE migration_name = '20260414130000_add_approval_workflow'
  AND finished_at IS NOT NULL;
-- Esperado: 1 row
```

## 2. Verificar enum ReportStatus contém 'aprovado' e 'rejeitado'

```sql
SELECT enumlabel
FROM pg_enum
WHERE enumtypid = (SELECT oid FROM pg_type WHERE typname = 'ReportStatus')
ORDER BY enumsortorder;
-- Esperado: rascunho, assinado, aprovado, rejeitado, cancelado
```

## 3. Verificar novos campos em slaughter_reports

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'slaughter_reports'
  AND column_name IN ('statusHistory', 'assignedToId', 'assignedAt')
ORDER BY column_name;
-- Esperado:
-- assignedAt    | timestamp without time zone | YES
-- assignedToId  | uuid    | YES
-- statusHistory | jsonb   | YES
```

## 4. Verificar novos campos em production_reports

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'production_reports'
  AND column_name IN ('statusHistory', 'assignedToId', 'assignedAt')
ORDER BY column_name;
-- Esperado: mesmos 3 campos
```

## 5. Verificar novos campos em shipping_reports

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'shipping_reports'
  AND column_name IN ('statusHistory', 'assignedToId', 'assignedAt')
ORDER BY column_name;
-- Esperado: mesmos 3 campos
```

## 6. Verificar foreign keys de assignedToId

```sql
SELECT tc.table_name, tc.constraint_name
FROM information_schema.table_constraints tc
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND tc.constraint_name LIKE '%assignedToId%'
ORDER BY tc.table_name;
-- Esperado: 3 rows (slaughter_reports, production_reports, shipping_reports)
```

## 7. Verificar que relatórios existentes não foram afetados

```sql
SELECT status, count(*)
FROM slaughter_reports
GROUP BY status;
-- Esperado: apenas rascunho/assinado/cancelado (sem aprovado/rejeitado)
-- statusHistory, assignedToId, assignedAt devem ser NULL para todos
```
