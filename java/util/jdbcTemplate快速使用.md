有些特殊时候想快速操作数据库，JdbcTemplate可避免繁琐配置，使用非常方便。示例代码如下：

```java
    public static void main(String[] args) {
        RowMapper<Data> rm = BeanPropertyRowMapper.newInstance(Data.class);
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        DataSource dataSource = DataSourceBuilder.create()
                .driverClassName("com.mysql.cj.jdbc.Driver")
                .url("jdbc:mysql://127.0.0.1:3306/test")
                .username("root")
                .password("password")
                .build();
        jdbcTemplate.setDataSource(dataSource);

        int count = jdbcTemplate.queryForObject("select count(1) from video20181017 where status = 0",Integer.class);
        System.out.println(count);
        for (int i = 0; i < count; i++) {
            Data data = jdbcTemplate.queryForObject("select * from video20181017 where status = 0 limit 1",rm);
            System.out.println(data);
            jdbcTemplate.update("update video20181017 set status = 1 where id = " + data.getId());
        }


    }
```