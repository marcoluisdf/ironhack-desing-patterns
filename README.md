# ironhack-desing-patterns

# Integrating Design Patterns in a Project Scenario


### Tasks
#### Design Problem Solving:

Scenario Description: Participants are provided with a series of common software design challenges. They will need to choose appropriate design patterns to solve these specific problems effectively.

Design Challenges:

Global Configuration Management: Design a system that ensures a single, globally accessible configuration object without access conflicts.

Dynamic Object Creation Based on User Input: Implement a system to dynamically create various types of user interface elements based on user actions.

State Change Notification Across System Components: Ensure components are notified about changes in the state of other parts without creating tight coupling.

Efficient Management of Asynchronous Operations: Manage multiple asynchronous operations like API calls which need to be coordinated without blocking the main application workflow.

Task: Outline solutions that integrate these patterns into a cohesive design to address the challenges.

Project Execution Simulation:

Simulate the application of these patterns in a hypothetical software project. Document the approach, rationale, and integration process of the chosen patterns as they apply to the design challenges.




## Singlenton:

Generamos un objeto que se encargara de gestionar de manera unica el acceso a la BD

``` java 
public class DatabaseConnection {
    private static DatabaseConnection instance;
    private String url = "jdbc:mysql://localhost:3306/mydatabase";
    private String username = "root";
    private String password = "123";

    private DatabaseConnection() throws SQLException {
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            this.connection = DriverManager.getConnection(url, username, password);
        } catch (ClassNotFoundException ex) {
            throw new SQLException(ex);
        }
    }

    public static DatabaseConnection getInstance() throws SQLException {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }

}
```

## State Change Notification Across System Components and Factory

Generamos un grupo de clases, que representaran a las divisas MXn y USD cada una contendra una logica personalisada para consultar ordenes pendientes a la BD mediante el singlenton que BD

Para posterior ejecutar ordenes de compra en un componenete externo
``` java
public interface Currency {
    void consultBuyOrders();
    void executeBuyOrder();
}

// ================================================================================================
public class MXNCurrency implements Currency {
    @Override
    public void consultBuyOrders() {
        // Logica para consultar ordenes pendientes de la divisa MXN
    }

    @Override
    public void executeBuyOrder() {
        // Lógica para ejecutar la orden de compra en un componente externo con procesos asyncronos
        System.out.println("Executing MXN buy order in external component.");
    }
}

// ================================================================================================
public class USDCurrency implements Currency {
    @Override
    public void consultBuyOrders() {
        // Logica para consultar ordenes pendientes de la divisa USD
    }

    @Override
    public void executeBuyOrder() {
        // Lógica para ejecutar la orden de compra en un componente externo con procesos asyncronos
        System.out.println("Executing USD buy order in external component.");
    }
}

// ================================================================================================

// Factory que estara implementada en el observer, que recibira cambios en los precios de las divisas

public class CurrencyFactory {
    public static Currency getCurrency(String currencyType) {
        if (currencyType == null) {
            return null;
        }
        if (currencyType.equalsIgnoreCase("MXN")) {
            return new MXNCurrency();
        } else if (currencyType.equalsIgnoreCase("USD")) {
            return new USDCurrency();
        }
        return null;
    }
}

// ================================================================================================
// Observer que notificar los cambios en las divisas al resto de componentes y poder ejecutar ordenes de compra
interface Observer {
    void update(String currency, double price);
}

public class CurrencyPriceObserver implements Observer {
    private String observerName;

    public CurrencyPriceObserver(String observerName) {
        this.observerName = observerName;
    }

    @Override
    public void update(String currency, double price) {
        System.out.println(observerName + " received update: " + currency + " price changed to " + price);
        buy(currency);
    }

    public void buy(String currency) {
        Currency currencyObj = CurrencyFactory.getCurrency(currency);
        if (currencyObj != null) {
            currencyObj.executeBuyOrder();
        } else {
            System.out.println("No currency found for: " + currency);
        }
    }
}
// ================================================================================================

public class CurrencyPrice {
    private List<Observer> observers;
    private String currency;
    private double price;
    private String cryptoOrder;

    public CurrencyPrice() {
        observers = new ArrayList<>();
    }

    public void setPrice(String currency, double price) {
        this.currency = currency;
        this.price = price;
        notifyObservers();
    }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(currency, price);
        }
    }

}

// ================================================================================================

 public class Main {
    public static void main(String[] args) {
        Observer observer1 = ObserverFactory.getObserver("CURRENCY_PRICE", "Observer 1");

        CurrencyPrice currencyPrice = new CurrencyPrice();
        currencyPrice.addObserver(observer1);

        currencyPrice.setPrice("USD", 19.50);
        currencyPrice.setPrice("MXN", 0.05);
        
    }
}

```


Simulación de ejecución de proyectos:

Singleton: Se utilizo para tener un unico componente que pueda gestionar el acceso concurrente a la BD

Factory Method: Se implemento un grupo de clases que representara a las divisas MXN y USD, cada una con su logica para consulktar ordenes pendiente de compra y poder ejecutar estas en componentes externos, mediante promesas

Observer: Se utilizó el patrón Observer para la notificación de cambio de estado, para este caso el precio de las divisas y se pudan ejecutar multiples ordenes de compra.

