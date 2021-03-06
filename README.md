# dubbo-web-demo
A Java web application with Dubbo as RPC framework &amp; ZooKeeper as its registry center yet ***WITHOUT** Spring*.

Nowadays, more and more Java applications see the Spring Framework as one of their must-depend libraries. The framework itself claims to be lightweight while sometimes our applications ought to be lighter, by "lighter" I mean "without it". Thus, I write this demo to explore the world without Spring which might be a better place, especially when each microservice is too simple to wear even that "lightweight" armor.
(However, to be accurate, Dubbo itself depends on Spring. SO if there is a replacement for Dubbo, this demo will be ***WITHOUT*** Spring entirely.)

I hope this could be enlightening in a way.

## Setup Dubbo Services without Spring

### Provider

**AccountApplication.java**
```java
/* setup AccountService provider */
DubboBootstrap bootstrap = DubboBootstrap.getInstance();
ServiceConfig<AccountService> service = new ServiceConfig<>();
service.setInterface(AccountService.class);
service.setRef(new AccountServiceImpl());
ApplicationConfig application = new ApplicationConfig("account-service");
application.setQosPort(22221);
bootstrap.application(application);
bootstrap.registry(new RegistryConfig(ZooKeeperConstant.zkServers));
bootstrap.protocol(new ProtocolConfig("dubbo", 20881));
bootstrap.service(service);
bootstrap.start();
```
### Consumer

**IndentServiceImpl.java**
```java
/* setup application & registry configuration */
DubboBootstrap bootstrap = DubboBootstrap.getInstance();
bootstrap.application(new ApplicationConfig("IndentService"));
bootstrap.registry(new RegistryConfig(ZooKeeperConstant.zkServers));

/* setup AccountService consumer instance */
accountService = DubboTool.getInstance(AccountService.class, bootstrap);

/* setup MerchandiseService consumer instance */
merchandiseService = DubboTool.getInstance(MerchandiseService.class, bootstrap);
```

**DubboTool.java**
```java
public class DubboTool {
    public static <T> T getInstance(Class<T> clazz, DubboBootstrap bootstrap) {
        ReferenceConfig<T> reference = new ReferenceConfig<>();
        reference.setBootstrap(bootstrap);
        reference.setInterface(clazz);
        reference.setCheck(false);
        return reference.get();
    }
}
```

## Setup HTTP Server without Spring

**BusinessServer.java**
```java
public class BusinessServer {
    private static final Logger LOGGER = LoggerFactory.getLogger(BusinessServer.class);

    private final String host;
    private final int port;

    private final HttpServer server;

    public BusinessServer(String host, int port) throws IOException {
        this.host = host;
        this.port = port;
        server = HttpServer.create(new InetSocketAddress(host, port), 0);
        initContext();
    }

    private void initContext() {
        server.createContext("/account", new AccountHandler());
        server.createContext("/merchandise", new MerchandiseHandler());
        server.createContext("/indent", new IndentHandler());
        server.createContext("/business", new BusinessHandler());
    }

    public void start() {
        server.start();
        LOGGER.info("server started at {}:{}", host, port);
    }

    public void stop() {
        server.stop(5);
        LOGGER.info("server stopped");
    }
}
```

**BusinessApplication.java**
```java
BusinessServer server = new BusinessServer("localhost", 8080);
server.start();
```
