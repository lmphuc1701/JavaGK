//create unique constraints
CREATE CONSTRAINT unique_supplier IF NOT EXISTS FOR (s:Supplier) REQUIRE s.supplier_id IS UNIQUE;
CREATE CONSTRAINT unique_product IF NOT EXISTS FOR (p:Product) REQUIRE p.product_id IS UNIQUE;
CREATE CONSTRAINT unique_order IF NOT EXISTS FOR (o:Order) REQUIRE o.order_id IS UNIQUE;

public class Neo4jconManager implements AutoCloseable {
    private final Driver driver;
    private final String dbName;

    public Neo4jconManager (String url, String user, String password, String dbName){
        this.driver = GraphDatabase.driver(url, AuthTokens.basic(user, password));
        this.dbName = dbName;
    }

    public Session openSession(){
        return driver.session(SessionConfig.forDatabase(dbName));
    }

    @Override
    public void close() {
        driver.close();
    }
}

public class Main {
    public static void main(String[] args) {
        String url = "bolt://localhost:7687";
        String user = "neo4j";
        String password = "Phuc170125.";
        String dbName = "phuc";

        Neo4jconManager manager = new Neo4jconManager(url, user, password, dbName);

        //Ket noi du lieu
        try (Session session = manager.openSession()) {
            Result result = session.run("RETURN 1 AS CHECK");

            if (result.hasNext()){
                System.out.println("Ket noi thanh cong");
            } else {
                System.out.println("Ket noi khong thanh cong");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        //Cau a
        //Cau b
        //Cau c
    }
}
