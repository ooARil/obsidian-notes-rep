
Все основные концепции Spring Batch:
* Job
* Step
* Reader
* Proccessor
* Writer
* Chunk
Описаны в этой документации: 

https://docs.spring.io/spring-batch/docs/current/reference/html/step.html

----
JdbcPagingItemReader:
```Java
public class MyJdbcPagingItemReader {

    private DataSource dataSource;

    public MyJdbcPagingItemReader(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public JdbcPagingItemReader<MyEntity> reader() throws Exception {

        Map<String, Order> sortKeys = new HashMap<>();
        sortKeys.put("id", Order.ASCENDING);

        SqlPagingQueryProviderFactoryBean queryProvider = new SqlPagingQueryProviderFactoryBean();
        queryProvider.setDataSource(dataSource);
        queryProvider.setSelectClause("select id, name, description");
        queryProvider.setFromClause("from my_table");
        queryProvider.setSortKeys(sortKeys);

        JdbcPagingItemReader<MyEntity> reader = new JdbcPagingItemReader<>();
        reader.setDataSource(dataSource);
        reader.setFetchSize(1000);
        reader.setRowMapper(new BeanPropertyRowMapper<>(MyEntity.class));
        reader.setQueryProvider(queryProvider.getObject());

        return reader;
    }
}
```
1. Сначала он отправляет SQL-запрос в базу данных для получения первой страницы данных. Размер страницы определяется параметром fetchSize.
2. Затем он читает данные из этой страницы и передает их в ItemProcessor для обработки.
3. После обработки всех данных на странице он отправляет еще один SQL-запрос в базу данных для получения следующей страницы данных.
4. Этот процесс повторяется до тех пор, пока не будут прочитаны все данные.

Данный reader при каждом вызове метода read() в chunk(), он открывает транкзацию.

Недостатки:
* Приходиться каждый раз сортировать значения, чтобы сдвигался offset. (Если нету индекса на это поле, - то этот тип ридера не подходит)

---
JdbcCursorItemReader:
```Java
@Configuration
public class MyBatchConfiguration {

    @Bean
    public JdbcCursorItemReader<MyData> myDataReader(DataSource dataSource) {
        JdbcCursorItemReader<MyData> reader = new JdbcCursorItemReader<>();
        reader.setDataSource(dataSource);
        reader.setSql("SELECT column1, column2, column3 FROM my_table");
        reader.setRowMapper(new MyDataRowMapper());
        return reader;
    }

    private static class MyDataRowMapper implements RowMapper<MyData> {
        @Override
        public MyData mapRow(ResultSet rs, int rowNum) throws SQLException {
            MyData data = new MyData();
            data.setColumn1(rs.getString("column1"));
            data.setColumn2(rs.getString("column2"));
            data.setColumn3(rs.getString("column3"));
            return data;
        }
    }
}
```
1. При создании экземпляра JdbcCursorItemReader, вы задаете SQL-запрос, который будет использоваться для извлечения данных из базы данных. 

2. Когда Spring Batch запускает Job, он вызывает метод open() JdbcCursorItemReader. Этот метод выполняет SQL-запрос и открывает курсор на результате.

3. Затем Spring Batch вызывает метод read() JdbcCursorItemReader для чтения данных. Этот метод извлекает следующую строку из курсора и возвращает ее.

4. Spring Batch продолжает вызывать метод read() до тех пор, пока он не вернет null (что означает, что все строки были прочитаны), или пока не будет достигнут размер chunk'а.

5. Когда размер chunk'а достигнут, Spring Batch передает этот chunk на обработку ItemProcessor'у (если он есть) и затем ItemWriter'у.

6. После записи chunk'а, Spring Batch автоматически коммитит транзакцию.

7. Этот процесс повторяется, пока все строки не будут прочитаны из курсора.

Данный reader создает только одну транкзацию при открытии курсора на весь step.

Недостатки:
* Долгая висящая транкзация, которая блокирует такие функции как autovacum PostgreSQL, что приводит к деградации таблицы. Т.к. не обновляется статистика по таблице, и проектировщик запроса PostgreSQL выбирает не оптимальный тип запроса.
------------
