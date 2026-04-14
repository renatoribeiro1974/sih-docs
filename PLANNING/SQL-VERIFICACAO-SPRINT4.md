# SQLs de Verificação — Sprint 4 (TASK-06 + TASK-09)

> Migration: `20260414140000_add_notifications_and_attachments`

---

## 1. Verificar migration aplicada

```sql
SELECT migration_name, finished_at
FROM _prisma_migrations
WHERE migration_name = '20260414140000_add_notifications_and_attachments'
  AND finished_at IS NOT NULL;
-- Esperado: 1 row
```

## 2. Verificar tabela notifications

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'notifications'
ORDER BY ordinal_position;
-- Esperado: id, recipientId, type, title, message, metadata, readAt, createdAt
```

## 3. Verificar tabela report_attachments

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'report_attachments'
ORDER BY ordinal_position;
-- Esperado: id, shippingReportId, fileName, fileKey, fileSize, mimeType, category, uploadedById, createdAt
```

## 4. Verificar foreign keys

```sql
SELECT tc.table_name, tc.constraint_name, ccu.table_name AS references_table
FROM information_schema.table_constraints tc
JOIN information_schema.constraint_column_usage ccu ON tc.constraint_name = ccu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND tc.table_name IN ('notifications', 'report_attachments')
ORDER BY tc.table_name;
-- Esperado: notifications → supervisor_profiles, report_attachments → shipping_reports + supervisor_profiles
```

## 5. Verificar indices

```sql
SELECT indexname, tablename
FROM pg_indexes
WHERE tablename IN ('notifications', 'report_attachments')
ORDER BY tablename;
-- Esperado: indices em recipientId+readAt, createdAt, shippingReportId
```
