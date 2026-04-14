# SQLs de Verificação — Sprint 1 (TASK-02 + TASK-03)

> Migration: `20260414120000_add_controlador_role_and_fields`
> Executar contra o banco `sih` após aplicar a migration.

---

## 1. Verificar que a migration foi aplicada

```sql
SELECT migration_name, finished_at
FROM _prisma_migrations
WHERE migration_name = '20260414120000_add_controlador_role_and_fields'
  AND finished_at IS NOT NULL;
-- Esperado: 1 row
```

## 2. Verificar enum UserRole contém 'controlador'

```sql
SELECT enumlabel
FROM pg_enum
WHERE enumtypid = (SELECT oid FROM pg_type WHERE typname = 'UserRole')
ORDER BY enumsortorder;
-- Esperado: admin, coordenador, supervisor, operador, controlador
```

## 3. Verificar novos campos em supervisor_profiles

```sql
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'supervisor_profiles'
  AND column_name IN ('companyGroup', 'extraPlantAccess', 'isManager')
ORDER BY column_name;
-- Esperado:
-- companyGroup     | text    | YES | null
-- extraPlantAccess | jsonb   | YES | null
-- isManager        | boolean | NO  | false
```

## 4. Verificar novo campo em plants

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'plants'
  AND column_name = 'companyGroup';
-- Esperado: companyGroup | text | YES
```

## 5. Teste funcional: criar controlador

```sql
-- Apenas para verificação, não executar em produção sem rollback planejado
-- INSERT INTO supervisor_profiles (id, "externalUserId", name, email, role, "companyGroup", "isManager")
-- VALUES (
--   uuid_generate_v4(),
--   'test-ctrl-001',
--   'Controlador Teste',
--   'ctrl-teste@fambras.org.br',
--   'controlador',
--   'IN',
--   false
-- );
-- Esperado: sucesso (role aceita 'controlador', companyGroup aceita 'IN')
```

## 6. Verificar que plantas existentes não foram afetadas

```sql
SELECT id, name, "sifCode", "companyGroup"
FROM plants
LIMIT 10;
-- Esperado: companyGroup = NULL para todas (ainda não preenchido)
-- Pendência: admin precisa preencher companyGroup (IN/IND) para cada planta
```
