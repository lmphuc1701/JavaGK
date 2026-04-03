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
        ProductDao pDao = new ProductDao(manager);
        List<Product> p = pDao.listProductsBySupplier("Pavlova Ltd.");

        System.out.println("------------------");
        System.out.println("Danh sach san pham");
        System.out.println(p);
        System.out.println("------------------");

        
        //Cau b
        //Cau c
    }
}

//DAO
public class ProductDAO {
    private  final Neo4jconnManager manager;

    public ProductDAO(Neo4jconnManager manager){
        this.manager = manager;
    }

    public List<Product> listProductsBySupplier (String companyName){
        if (companyName == null || companyName.trim().isEmpty()){
            throw new IllegalArgumentException("khong duoc rong");
        }

        List<Product> productList = new ArrayList<>();

        String query = "MATCH(s:Supplier {company_name: $name})-[r]-(p:Product) "+
                "RETURN p";

        try(Session session = manager.openSession() ){
            Result result = session.run(query, Map.of(
                    "name", companyName
            ));

            while (result.hasNext()){
                Node node = result.next().get("p").asNode();
                Product p = new Product();
                p.setUnit(node.get("unit").asString());
                p.setUnitsInStock(node.get("units_in_stock").asInt());
                p.setProductID(node.get("product_id").asString());
                p.setUnitPrice(node.get("unit_price").asDouble());
                p.setProductName(node.get("product_name").asString());

                productList.add(p);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return productList;
    }

}
