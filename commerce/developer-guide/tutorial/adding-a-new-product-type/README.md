# Adding a New Product Type

This tutorial will show you how to add a new product type by implementing three interfaces: [CPType](https://github.com/liferay/com-liferay-commerce/blob/2.0.4/commerce-product-api/src/main/java/com/liferay/commerce/product/type/CPType.java), [ScreenNavigationCategory](https://github.com/liferay/liferay-portal/blob/7.1.3-ga4/modules/apps/frontend-taglib/frontend-taglib/src/main/java/com/liferay/frontend/taglib/servlet/taglib/ScreenNavigationCategory.java), and [ScreenNavigationEntry](https://github.com/liferay/liferay-portal/blob/7.1.3-ga4/modules/apps/frontend-taglib/frontend-taglib/src/main/java/com/liferay/frontend/taglib/servlet/taglib/ScreenNavigationEntry.java).

A product type is a categorization of products that may have different types of information, and may even need to be presented differently. Liferay Commerce provides three product types out of the box: [Simple](https://github.com/liferay/com-liferay-commerce/blob/2.0.4/commerce-product-type-simple/src/main/java/com/liferay/commerce/product/type/simple/internal/SimpleCPType.java), [Grouped](https://github.com/liferay/com-liferay-commerce/blob/2.0.4/commerce-product-type-grouped-web/src/main/java/com/liferay/commerce/product/type/grouped/web/internal/GroupedCPType.java), and [Virtual](https://github.com/liferay/com-liferay-commerce/blob/2.0.4/commerce-product-type-virtual-web/src/main/java/com/liferay/commerce/product/type/virtual/web/internal/VirtualCPType.java).

![Out of the box product types](./images/01.png "Out of the box product types")

## Overview

1. **Deploy an Example**
1. **Walk Through the Example**
1. **Additional Information**

## Deploy an Example

In this section, we will get an example product type up and running on your instance of Liferay Commerce. Follow these steps:

1. Start Liferay Commerce.

    ```bash
    docker run -it -p 8080:8080 liferay/commerce:2.0.4
    ```

1. Download and unzip [Acme Commerce Product Type](./liferay-c1n4.zip).

    ```bash
    curl liferay-c1n4.zip
    ```

    ```bash
    unzip liferay-c1n4.zip
    ```

1. Go to `c1n4-impl`.

    ```bash
    cd c1n4-impl
    ```

1. Build and deploy the example.

    ```bash
    ./gradlew deploy -Ddeploy.docker.container.id=$(docker ps -lq)
    ```

    >**Note:** This command is the same as copying the deployed jars to /opt/liferay/osgi/modules on the Docker container.

1. Confirm the deployment in the Liferay Docker container console.

    ```bash
    STARTED com.acme.c1n4.impl_1.0.0
    ```

1. Verify that the example product type was added. Open your browser to `https://localhost:8080` and navigate to _Control Panel_ → _Commerce_ → _Products_. Then, click on the + icon to add a new product. The new product type ("Example") will be present in the list of types to choose from.

![New product type](./images/02.png "New product type")

Next, let's dive deeper to learn more.

## Walk Through the Example

In this section, we will review the example we deployed. First, we will annotate two classes for OSGi registration. Second, we will implement the three interfaces: `CPType`, `ScreenNavigationCategory`, and `ScreenNavigationEntry`. And third, we will complete our implementation of these interfaces.

### Annotate the Classes for OSGi Registration

We will create two classes: a class for the product type itself, and a screen navigation entry class for a custom screen.

#### Annotate the Product Type Class

```java
@Component(
    immediate = true,
    property = {
        "commerce.product.type.display.order:Integer=21",
        "commerce.product.type.name=" + C1N4CPType.NAME
    },
    service = CPType.class
)
public class C1N4CPType implements CPType {

    public static final String NAME = "Example";
```

> Our product type class implements the `CPType` interface.
>
> The product type name must be a unique value so that Liferay Commerce can distinguish our product type from existing product types.
>
> The `commerce.product.type.display.order` value indicates how far into the list of product types our product type will appear in the UI.

#### Annotate the Screen Navigation Entry Class

```java
@Component(
    property = {
        "screen.navigation.category.order:Integer=11",
        "screen.navigation.entry.order:Integer=11"
    },
    service = {ScreenNavigationCategory.class, ScreenNavigationEntry.class}
)
public class C1N4ScreenNavigationEntry
    implements ScreenNavigationCategory, ScreenNavigationEntry<CPDefinition> {

    public static final String KEY = "Example";
```

> Our screen navigation entry class implements both the `ScreenNavigationCategory` and `ScreenNavigationEntry` interfaces.
>
> It is important to provide a distinct key for the navigation screen class so that Liferay Commerce can distinguish it as a separate screen from the existing screens. Reusing a key that is already in use will override the existing associated navigation screen.
>
> The `screen.navigation.category.order` and `screen.navigation.entry.order` values together determine what position in the product type screens this screen will appear. For example, [the Details screen class](https://github.com/liferay/com-liferay-commerce/blob/2.0.4/commerce-product-definitions-web/src/main/java/com/liferay/commerce/product/definitions/web/internal/servlet/taglib/ui/CPDefinitionDetailsScreenNavigationEntry.java) has these values set to 10; setting them to 11 will ensure that our custom screen appears after it in the list.

### Review the Interfaces for the Two Classes

[Preamble?]

#### Review the `CPType` Interface

Implement the following methods of `CPType` in the product type class:

```java
public void deleteCPDefinition(long cpDefinitionId) throws PortalException;
```

> This method is where any custom deletion logic for the product type will be added.

```java
public String getLabel(Locale locale);
```

> This returns a text label that describes the product type. See the implementation in [C1N4CPType.java](./liferay-c1n4.zip/c1n4-impl/src/main/java/com/acme/c1n4/internal/commerce/product/type/C1N4CPType.java) for a reference in retrieving the label with a language key.

```java
public String getName();
```

> This returns the name of our product type. This name may be a language key that corresponds to the name that will appear in the UI.

#### Review the `ScreenNavigationCategory` Interface

Implement the following methods in the screen navigation entry class:

```java
String getCategoryKey();
```

> This returns a unique identifier for the category used for the screen navigation entry.

```java
String getLabel(Locale var1);
```

> This returns a text label for the screen navigation entry that will be displayed in the UI. See the implementation in [C1N4CPTypeScreenNavigationEntry.java](./liferay-c1n4.zip/c1n4-impl/src/main/java/com/acme/c1n4/internal/commerce/product/type/C1N4CPTypeScreenNavigationEntry.java) for a reference in retrieving the label with a language key.

```java
String getScreenNavigationKey();
```

> This returns a key to indicate where our screen should appear in Liferay. Return the value `cp.definition.general` so it properly appears among the other screens for products.

#### Review the `ScreenNavigationEntry` Interface

Implement the following methods in the screen navigation entry class:

```java
String getCategoryKey();
```

> This returns a unique identifier for the screen navigation category used by our screen.

```java
String getEntryKey();
```

> This returns a unique identifier for the screen navigation entry. It may return the same value as `getCategoryKey`.

```java
String getScreenNavigationKey();
```

> This is the same method as `getScreenNavigationKey` for the `ScreenNavigationCategory` interface. We have already implemented this method by returning `cp.definition.general`.

```java
void render(
        HttpServletRequest var1, HttpServletResponse var2)
    throws IOException;
```

> This will be where we add the code to render a customized screen for our product type.

### Complete the Product Type

The product type is comprised of backend logic for deleting the product, logic to render the screen in the navigation menu, and the custom screen itself. Do the following:

* [Configure the `ServletContext` for the module.](#configure-the-servletcontext-for-the-module)
* [Implement the `ScreenNavigationEntry`'s `render` method.](#implement-the-screennavigationentrys-render-method)
* [Override the `ScreenNavigationEntry`'s `isVisible` method.](#override-the-screennavigationentrys-isvisible-method)
* [Add the product type deletion logic to `deleteCPDefinition`.](#add-the-product-type-deletion-logic-to-deletecpdefinition)
* [Add a JSP to render the custom screen.](#add-a-jsp-to-render-the-custom-screen)
* [Add the language key to `Language.properties`](#add-the-language-key-to-languageproperties)

#### Configure the `ServletContext` for the Module

Define the `ServletContext` in our `ScreenNavigationEntry` class using the symbolic name of our bundle so that it can find the JSP in our module:

```java
@Reference(target = "(osgi.web.symbolicname=com.acme.c1n4.impl)")
private ServletContext _servletContext;
```

> The value we set for `osgi.web.symbolicname` matches the value for `Bundle-SymbolicName` in our [bnd.bnd file](./liferay-c1n4.zip/c1n4-impl/bnd.bnd). These values must match for the `ServletContext` to locate the JSP.
>
> We also need to declare a unique value for `Web-ContextPath` in our bnd.bnd file so the `ServletContext` is correctly generated. In our example, `Web-ContextPath` is set to `/commerce-product-type`. See [bnd.bnd](./liferay-c1n4.zip/c1n4-impl/bnd.bnd) for a reference on these values.

#### Implement the `ScreenNavigationEntry`'s `render` Method

```java
@Override
public void render(
        HttpServletRequest httpServletRequest,
        HttpServletResponse httpServletResponse)
    throws IOException {

    _jspRenderer.renderJSP(
        _servletContext, httpServletRequest, httpServletResponse,
        "/edit_product.jsp");
}
```

> Use a `JSPRenderer` to render the JSP for our product type's custom screen (in our example, [edit_product.jsp](./liferay-c1n4.zip/c1n4-impl/src/main/resources/META-INF/resources/edit_product.jsp)). Provide the `ServletContext` as a parameter to find the JSP we have created.

#### Override the `ScreenNavigationEntry`'s `isVisible` Method

```java
@Override
public boolean isVisible(User user, CPDefinition context) {
    if (context == null) {
        return false;
    }

    String productTypeName = context.getProductTypeName();

    if (productTypeName.equals(getCategoryKey())) {
        return true;
    }

    return false;
}
```

> Implement logic here to determine when to show the custom screen is visible. In our example, we only check whether the product type from the `CPDefinition` matches our example product type.

#### Add the Product Type Deletion Logic to `deleteCPDefinition`

Our example does not require any logic to be added to `deleteCPDefinition`.

#### Add a JSP to Render the Custom Screen

In our example, we are adding a JSP with only a text field.

```jsp
<%@ taglib uri="http://liferay.com/tld/aui" prefix="aui" %>

<aui:input name="exampleInput" type="text" />
```

> Implement any other inputs or actions desired on the custom screen here, such as a form or MVC action command. See [MVC Action Command](https://portal.liferay.dev/docs/7-1/tutorials/-/knowledge_base/t/mvc-action-command) for more information on adding an MVC action command that can be accessed from the JSP.

#### Add the Language Key to `Language.properties`

Add the language key and its value to a [Language.properties](./liferay-c1n4.zip/c1n4-impl/src/main/resources/content/Language.properties) file within our module:

```
example=Example
```

> See [Localizing Your Application](https://help.liferay.com/hc/en-us/articles/360018168251-Localizing-Your-Application) for more information.

## Conclusion

Congratulations! You now know the basics for implementing the `CPType` interface, and have implemented a new product type with its own custom screen to Liferay Commerce.

## Additional Information

* [Introduction to Product Types](../../../user-guide/catalog/creating-and-managing-products/product-types/introduction-to-product-types)
* [Localizing Your Application](https://help.liferay.com/hc/en-us/articles/360018168251-Localizing-Your-Application)
