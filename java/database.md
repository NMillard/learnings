# Java + Database

Interacting with any database in Java (with maven) is to install a database driver, by taking a dependency on a package.

For postgres, you can grab this package:

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.3</version>
</dependency>
```

## Querying data
Let's see how you can connect an query a database, without using anything else than the driver manager (`DriverManager` class).

For instance, say we have a `WorkFlow` class and you want to get all rows.

```java
// WorkFlow definition at bottme of this snippet.

var workflows = new ArrayList<Workflow>();
String sql = "SELECT * FROM workflow.workflows";

try (Connection connection = DriverManager.getConnection("jdbc:postgresql://host:port/db", "postgres", "postgres");
     Statement statement = connection.createStatement();
     ResultSet set = statement.executeQuery(sql)) {
    while (set.next()) {
        var workflow = new Workflow(
                UUID.fromString(set.getString("id")),
                set.getString("name"),
                set.getString("type"));

        workflows.add(workflow);
    }
}

public class Workflow {
    public Workflow(UUID id, String name, String type) {
        this.id = id;
        this.name = name;
        this.type = type;
    }

    private UUID id;
    private String name;
    private String type;

    // getters
}
```