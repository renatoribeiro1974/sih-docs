# SQLs de Verificação — Sprint 5 (TASK-11)

> Migration: `20260414150000_add_halal_cert_fields`

---

## 1. Verificar migration aplicada

```sql
SELECT migration_name, finished_at
FROM _prisma_migrations
WHERE migration_name = '20260414150000_add_halal_cert_fields'
  AND finished_at IS NOT NULL;
-- Esperado: 1 row
```

## 2. Verificar campos em meat_inventory_receipts

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'meat_inventory_receipts'
  AND column_name IN ('halalCertData', 'halalCertExpiryDate', 'halalCertSource')
ORDER BY column_name;
-- Esperado: 3 campos (jsonb, date, text — todos nullable)
```

## 3. Verificar campos em batch_inventories

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'batch_inventories'
  AND column_name IN ('halalCertNumber', 'halalCertData', 'halalCertExpiryDate', 'halalCertSource')
ORDER BY column_name;
-- Esperado: 4 campos (text, jsonb, date, text — todos nullable)
```

## 4. Verificar campos em shipping_reports

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'shipping_reports'
  AND column_name IN ('halalCertData', 'halalCertSource')
ORDER BY column_name;
-- Esperado: 2 campos (jsonb, text — ambos nullable)
```

## 5. Verificar que dados existentes não foram afetados

```sql
SELECT count(*) FROM meat_inventory_receipts WHERE "halalCertSource" IS NOT NULL;
SELECT count(*) FROM batch_inventories WHERE "halalCertNumber" IS NOT NULL;
SELECT count(*) FROM shipping_reports WHERE "halalCertSource" IS NOT NULL;
-- Esperado: 0 para todos (campos novos, ainda não preenchidos)
```
