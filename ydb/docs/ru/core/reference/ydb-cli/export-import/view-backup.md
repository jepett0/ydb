# Особенности восстановления представлений из резервных копий

При восстановлении [представлений](../../../concepts/datamodel/view.md) из резервной копии запрос представления может быть автоматически изменён для сохранения корректности ссылок на объекты схемы. Восстановленные из резервной копии представления будут ссылаться на восстановленные таблицы, а не на ранее существовавшие в целевой базе таблицы. Взаимное расположение представлений и объектов, на которые они ссылаются, сохраняется таким, каким оно было во время создания резервной копии.

## Примеры

### Восстановление корня базы данных по тому же пути {#example-root-root}

Рассмотрим следующий сценарий:

1. Создаётся представление:

    ```sql
    CREATE VIEW root_view WITH security_invoker = TRUE AS
        SELECT * FROM root_table;
    ```

2. Создаётся резервная копия базы:

    ```bash
    ydb tools dump --path . --output ./my_backup
    ```

3. Выполняется очистка базы данных:

    ```bash
    ydb scheme rmdir --force --recursive .
    ```

4. База данных восстанавливается:

    ```bash
    ydb tools dump --path . --input ./my_backup
    ```

В результате описанных выше шагов представление `root_view` восстанавливается и читает из таблицы `root_table`:

```bash
ydb sql --script 'select * from root_view' --explain
```

В выводе выполненной команды отображается: `TableFullScan (Table: root_table, ...`

### Восстановление корня базы данных в подпапку {#example-root-subfolder}

Рассмотрим следующий сценарий:

1. Создаётся представление:

    ```sql
    CREATE VIEW my_view WITH security_invoker = TRUE AS
        SELECT * FROM my_table;
    ```

2. Создаётся резервная копия базы:

    ```bash
    ydb tools dump --path . --output ./my_backup
    ```

3. База данных восстанавливается в подпапку `a/b/c`:

    ```bash
    ydb tools restore --path a/b/c --input ./my_backup
    ```

В результате описанных выше шагов представление `a/b/c/my_view` восстанавливается и читает из таблицы `a/b/c/my_table`:

```bash
ydb sql --script 'select * from `a/b/c/my_view`' --explain
```

В выводе выполненной команды отображается: `TableFullScan (Table: a/b/c/my_table, ...`

### Восстановление подпапки в корень базы данных {#example-subfolder-root}

Рассмотрим следующий сценарий:

1. Повторяются шаги 1-3 предыдущего сценария [{#T}](#example-root-subfolder).
2. Создаётся резервная копия подпапки `a/b/c` базы данных:

    ```bash
    ydb tools dump --path a/b/c --output ./subfolder_backup
    ```

3. Выполняется очистка базы данных:

    ```bash
    ydb scheme rmdir --force --recursive .
    ```

4. Резервная копия подпапки восстанавливается в корень базы данных:

    ```bash
    ydb tools restore --path . --input ./subfolder_backup
    ```

В результате описанных выше шагов представление `my_view` восстанавливается и читает из таблицы `my_table`:

```bash
ydb sql --script 'select * from my_view' --explain
```

В выводе выполненной команды отображается: `TableFullScan (Table: my_table, ...`

### Восстановление корня базы данных в корень другой базы данных {#example-root-different-root}

Рассмотрим следующий сценарий:

1. Создаётся представление:

    ```sql
    CREATE VIEW root_view WITH security_invoker = TRUE AS
        SELECT * FROM root_table;
    ```

2. Создаётся резервная копия базы:

    ```bash
    ydb --endpoint <endpoint> --database /my_database tools dump --path . --output ./my_backup
    ```

    Обратите внимание на `--database /my_database` в строке подключения.

3. Резервная копия базы данных восстанавливается в другую базу данных:

    ```bash
    ydb --endpoint <endpoint> --database /restored_database tools dump --path . --input ./my_backup
    ```

    Обратите внимание на `--database /restored_database` в строке подключения.

В результате описанных выше шагов представление `root_view` восстанавливается и читает из таблицы `root_table`, расположенной в `/restored_database`:

```bash
ydb --endpoint <endpoint> --database /restored_database sql --script 'select * from root_view' --explain
```

В выводе выполненной команды отображается: `TableFullScan (Table: root_table, ...`
