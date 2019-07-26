# Using a Custom Inventory Engine

This tutorial will show you how to add a custom inventory engine by implementing the `CommerceInventoryEngine` interface.

Inventory engines manage the quantities of stocked products in your instance of Liferay Commerce. Liferay Commerce provides one default inventory engine, `CommerceInventoryEngineImpl`.

(Establishing Image Placeholder)

## Overview

1. **Deploy an Example**
1. **Walk Through the Example**
1. **Additional Information**

## Deploy an Example

In this section, we will get an example inventory engine up and running on your instance of Liferay Commerce. Follow these steps:

1. Start Liferay Commerce.

    ```bash
    docker run -it -p 8080:8080 liferay/commerce:2.0.1
    ```

1. Download and unzip the [Acme Commerce ...]() to your project directory.

    ```bash
    curl liferay-xxxx.zip
    ```

    ```bash
    unzip liferay-xxxx.zip
    ```

1. Go to `xxxx-impl`.

    ```bash
    cd xxxx-impl
    ```

1. Build and deploy the example.

    ```bash
    ./gradlew deploy -Ddeploy.docker.container.id=$(docker ps -lq)
    ```

    >Note: This command is the same as copying the deployed jars to /opt/liferay/osgi/modules on the Docker container.

1. Confirm the deployment in the Liferay Docker container console.

    ```bash
    STARTED com.acme.xxxx.internal.commerce.foo.method_1.0.0
    ```

1. Verify that the example ... was added. Open your browser to `https://localhost:8080` and navigate to ...

(Deployed Sample Image Placeholder)

Next, let's dive deeper to learn more.

## Walk Through the Example

In this section, we will take a more in-depth review of the example we deployed. First, we will annotate the class for OSGi registration; second we will implement the `CommerceInventoryEngine` interface; and third, we will implement some custom logic.

### Annotate Your Class for OSGi Registration

```java
@Component(immediate = true, service = CommerceInventoryEngine.class)
    public class ExampleCommerceInventoryEngine implements CommerceInventoryEngine {
```

### Implement the `CommerceInventoryEngine` Interface

The following six methods are required:

```java
public void consumeQuantity(long userId, long commerceInventoryWarehouseId, String sku, int quantity, long bookedQuantityId, Map<String, String> context) throws PortalException;
```

```java
public void decreaseStockQuantity(long commerceInventoryWarehouseId, String sku, int quantity) throws PortalException;
```

```java
public Map<String, Integer> getStockQuantities(long companyId, long channelGroupId, List<String> skus) throws PortalException;
```

```java
public int getStockQuantity(long companyId, long channelGroupId, String sku) throws PortalException;
```

```java
public int getStockQuantity(long companyId, String sku) throws PortalException;
```

```java
public void increaseStockQuantity(long userId, long commerceInventoryWarehouseId, String sku, int quantity) throws PortalException;
```

To better understand each of the required methods mentioned above, let's look at `[example class]`. We will review the implementation of each required method in sequence.

1. `public void ConsumeQuantity(long userId, long commerceInventoryWarehouseId, String sku, int quantity, long bookedQuantityId, Map<String, String> context) throws PortalException;`

    ```java
    @Override
    @Transactional(
        propagation = Propagation.REQUIRED, rollbackFor = Exception.class
    )
    public void consumeQuantity(
            long userId, long commerceInventoryWarehouseId, String sku, int quantity, long bookedQuantityId,
            Map<String, String> context)
        throws PortalException {

        if (bookedQuantityId > 0) {
            CommerceInventoryBookedQuantity commerceInventoryBookedQuantity =
                _commerceBookedQuantityLocalService.
                    getCommerceInventoryBookedQuantity(bookedQuantityId);

            _commerceBookedQuantityLocalService.consumeCommerceBookedQuantity(
                commerceInventoryBookedQuantity.
                    getCommerceInventoryBookedQuantityId(),
                quantity);

            decreaseStockQuantity(commerceInventoryWarehouseId, sku, quantity);
        }
        else {
            decreaseStockQuantity(commerceInventoryWarehouseId, sku, quantity);
        }

        String description =
            "Consume Quantity: " + _jsonFactory.serialize(context);

        _commerceInventoryAuditLocalService.addCommerceInventoryAudit(
            userId, sku, quantity, description);
    }
    ```
    > This method handles both decreasing the quantity of a stocked item when it is consumed, and updating the booked quantity, ...
    > First...
    >
    > It also adds an inventory audit for the consumed quantity, using `CommerceInventoryAuditLocalService`; for the implementation of this service, see [CommerceInventoryAuditLocalServiceImpl](https://raw.githubusercontent.com/liferay/com-liferay-commerce/7.1.x/commerce-inventory-service/src/main/java/com/liferay/commerce/inventory/service/impl/CommerceInventoryAuditLocalServiceImpl.java).

1. `public void decreaseStockQuantity(long commerceInventoryWarehouseId, String sku, int quantity) throws PortalException;`

    ```java
    @Override
    @Transactional(
        propagation = Propagation.REQUIRED, rollbackFor = Exception.class
    )
    public void decreaseStockQuantity(
            long commerceInventoryWarehouseId, String sku, int quantity)
        throws PortalException {

        CommerceInventoryWarehouseItem commerceInventoryWarehouseItem =
            _commerceInventoryWarehouseItemLocalService.
                fetchCommerceInventoryWarehouseItem(
                    commerceInventoryWarehouseId, sku);

        _commerceInventoryWarehouseItemLocalService.
            updateCommerceInventoryWarehouseItem(
                commerceInventoryWarehouseItem.
                    getCommerceInventoryWarehouseItemId(),
                commerceInventoryWarehouseItem.getQuantity() - quantity);
    }
    ```

    > This handles decreasing the stocked quantity of a consumed item. It does this by fetching and then updating the existing quantity through the `CommerceInventoryWarehouseItemLocalService`. For the implementation of that service, see [CommerceInventoryWarehouseItemLocalServiceImpl.java](https://raw.githubusercontent.com/liferay/com-liferay-commerce/7.1.x/commerce-inventory-service/src/main/java/com/liferay/commerce/inventory/service/impl/CommerceInventoryWarehouseItemLocalServiceImpl.java).
    >
    > Note that this method is normally called by `consumeQuantity()`.

1. `public Map<String, Integer> getStockQuantities(long companyId, long channelGroupId, List<String> skus);`

    ```java
    @Override
    public Map<String, Integer> getStockQuantities(
        long companyId, long channelGroupId, List<String> skus) {

        Map<String, Integer> results = new HashMap<>();

        for (String sku : skus) {
            int stockQuantity = getStockQuantity(
                companyId, channelGroupId, sku);

            results.put(sku, stockQuantity);
        }

        return results;
    }
    
    > This returns a map of stocked quantities for a list of items in a specific [channel group?], identified by their SKUs. It uses the `getStockedQuantity(long, long, String)` implementation to achieve this.

1. `public int getStockQuantity(long companyId, long groupId, String sku);`

    ```java
    @Override
    public int getStockQuantity(
        long companyId, long channelGroupId, String sku) {

        int stockQuantity =
            _commerceInventoryWarehouseItemLocalService.getStockQuantity(
                companyId, channelGroupId, sku);

        int commerceBookedQuantity =
            _commerceBookedQuantityLocalService.getCommerceBookedQuantity(sku);

        return stockQuantity - commerceBookedQuantity;
    }
    ```
    
    > This returns the quantity of a single item, identified by its SKU. The stock quantity returned is considered as the total stocked quantity, minus the booked quantity. It uses the `CommerceInventoryWarehouseItemLocalService` and `CommerceBookedQuantityLocalService` to get these quantities, respectively.
    >
    > Note that this method is called by `getStockQuantities()`.

1. `public int getStockQuantity(long companyId, String sku);`

    ```java
    @Override
    public int getStockQuantity(long companyId, String sku) {
        int stockQuantity =
            _commerceInventoryWarehouseItemLocalService.getStockQuantity(
                companyId, sku);

        int commerceBookedQuantity =
            _commerceBookedQuantityLocalService.getCommerceBookedQuantity(sku);

        return stockQuantity - commerceBookedQuantity;
    }
    ```
    
    > This returns the quantity of a single item, identified by its SKU, but without a specific group. It does this in a similar way to the overloaded method above.

1. `public void increaseStockQuantity(long userId, long commerceInventoryWarehouseId, String sku, int quantity) throws PortalException;

    ```java
    @Override
    @Transactional(
        propagation = Propagation.REQUIRED, rollbackFor = Exception.class
    )
    public void increaseStockQuantity(
            long userId, long commerceInventoryWarehouseId, String sku,
            int quantity)
        throws PortalException {

        CommerceInventoryWarehouseItem commerceInventoryWarehouseItem =
            _commerceInventoryWarehouseItemLocalService.
                fetchCommerceInventoryWarehouseItem(
                    commerceInventoryWarehouseId, sku);

        _commerceInventoryWarehouseItemLocalService.
            updateCommerceInventoryWarehouseItem(
                commerceInventoryWarehouseItem.
                    getCommerceInventoryWarehouseItemId(),
                commerceInventoryWarehouseItem.getQuantity() + quantity);

        _commerceInventoryAuditLocalService.addCommerceInventoryAudit(
            userId, sku, quantity, "Increase Quantity");
    }
    ```
    
    > This handles increasing the stocked quantity of an item, using the `CommerceInventoryWarehouseItemLocalService`. It also adds an inventory audit for the increased quantity, using `CommerceInventoryAuditLocalService`.

We also need to include the extra Liferay services and providers used by these methods. These ones should be enough to get us started for our inventory engine:

```java
@Reference
private CommerceInventoryBookedQuantityLocalService
    _commerceBookedQuantityLocalService;

@Reference
private CommerceInventoryAuditLocalService
    _commerceInventoryAuditLocalService;

@Reference
private CommerceInventoryWarehouseItemLocalService
    _commerceInventoryWarehouseItemLocalService;

@Reference
private JSONFactory _jsonFactory;
```

### Add Custom Logic
Custom logic can be included in any of the previously mentioned methods. In our example, we will be adding a simple warning message whenever stock is updated over a certain threshold.

```java
@Override
@Transactional(
    propagation = Propagation.REQUIRED, rollbackFor = Exception.class
)
public void increaseStockQuantity(
        long userId, long commerceInventoryWarehouseId, String sku,
        int quantity)
    throws PortalException {

    CommerceInventoryWarehouseItem commerceInventoryWarehouseItem =
        _commerceInventoryWarehouseItemLocalService.
            fetchCommerceInventoryWarehouseItem(
                commerceInventoryWarehouseId, sku);

    int updatedQuantity =
        commerceInventoryWarehouseItem.getQuantity() + quantity;

    _commerceInventoryWarehouseItemLocalService.
        updateCommerceInventoryWarehouseItem(
            commerceInventoryWarehouseItem.
                getCommerceInventoryWarehouseItemId(),
            updatedQuantity);

    if (updatedQuantity > WARN_THRESHOLD_QUANTITY && _log.isWarnEnabled()) {
        _log.warn(
            "Updated stock quantity of " + sku + " exceeds threshold: "
                + updatedQuantity);
    }

    _commerceInventoryAuditLocalService.addCommerceInventoryAudit(
        userId, sku, quantity, "Increase Quantity");
}
```
> To add our custom warning message, we need to add the logic into `increaseStockQuantity()`. We could also implement a custom method to achieve this, but we would still need to call it from one of the methods defined in `CommerceInventoryEngine`.

## Conclusion

Congratulations! You now know the basics for implementing the `CommerceInventoryEngine` interface, and have added a custom warning message for overstocked quantities.

## Additional Information
[Inventory Audits]()