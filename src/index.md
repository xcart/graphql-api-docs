---
title: "X-Cart GraphQL API Provider"
---

# GraphQL API Connector Documentation

**Table of contents**

- [Authentication](#authentication)
- [Module-related queries](#module-related-queries)
- [Examples](#examples)
- [External auth](#external-auth)
- [Checkout flow](#checkout-flow)
- [WebView implementation](#webview-implementation)
- [Checkout-related queries](#checkout-related-queries)

X-Cart 5 can be enhanced with the following add-ons to provide GraphQL compatible catalog API, suitable for making alternative frontend apps (e.g. mobile apps):

*   QSL\GraphQLApi (v5.4.1)
*   QSL\MobileApp (v5.4.0)

These add-ons don’t provide the capability to allow for an admin-level application yet but can be easily extended to add more queries and mutations.

The API endpoint is available at `//<hostname>/admin.php?target=graphql_api`

You can explore it via such solutions like [GraphiQL](https://github.com/graphql/graphiql) (also, Chrome extension [ChromeiQL](https://chrome.google.com/webstore/detail/chromeiql/fkkiamalmpiidkljmicmjfbieiclmeij)) or by using the compiled documentation [https://xcart.github.io/graphql-api-docs/query.doc.html](https://xcart.github.io/graphql-api-docs/query.doc.html). 

X-Cart also maintains a demo store which can be used to test the API connection: [https://mobile.x-cart.com/admin.php?target=graphql_api](https://mobile.x-cart.com/admin.php?target=graphql_api)

This demo store storefront can be accessed by the URL: [https://mobile.x-cart.com.](https://mobile.x-cart.com/)

The API can be easily integrated via such solutions like the whole universe of different Apollo Clients ([https://www.apollographql.com/](https://www.apollographql.com/)).


## Authentication

Request authentication is JWT-based. Most of the query objects can be accessed without any token - replicating how the website storefront can be accessed. User-related data, such as customer orders information, address and personal can only be accessed by providing the user JWT token.

Do not provide any token if you’re not sure if it’s the valid one. If invalid token is presented, the API won’t allow access to any information, even if it doesn’t require a token.

Any requests to protected areas should include a special GraphQL variable called `context`.

This variable should be an object of the following string fields: 



*   jwt: Client JWT token
*   cur: Currency 2-char code
*   lng: Language 2-char code


### Receiving token for registered user

The JWT can be received via the following mutation:


```
mutation Auth($auth: AuthInput!) {
  auth(auth: $auth)
}
```


Variables:


```
{
  "auth": {
    "login": "test@example.com",
    "password": "test"
  }
}
```


As you can see, $auth should be an object of the following string fields:



*   login
*   password

If the login \ password is correct, this request will return the similar output:


```
{
  "data": {
    "auth": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJ1c2VyX2lkIjo1ODYsImFjY2VzcyI6ImN1c3RvbWVyIiwiZXhwaXJlIjoiMjAxOS0wNi0xOFQxNTo0NDoxMyswNDowMCJ9.DrGeyReowpizC8UKQRuIK34-CtHXxglkPVOVfyn1FHft0udXKPYqIQJgPJ15jBylYvmcWa13JolICXTkJpuPuQ"
  }
}
```



### Receiving token for anonymous user

By default, no persistent data is being stored by the backend during API access, therefore there are no tokens and cart ids. However, right after a successful mutation that changes the cart to be non-empty (e.g. after adding the product to cart), the `cart` object will have a `token` param which can be used to continue making requests with a single session. Temporary anonymous user will be created on the backend.

For instance, you can receive a jwt after making such mutation:


```
mutation {
    addProductToCart(product_id: 7) {
        id
        token
        total_amount
    }
}
```


The result will contain a token to be used in the next requests:


```
{
    "data": {
        "addProductToCart": {
            "id": "119",
            "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzUxMiJ9.eyJ1c2VyX2lkIjoyMjAsImFjY2VzcyI6ImFub255bW91cyIsImV4cGlyZSI6IjIwMjAtMDMtMjZUMDc6MTA6MTIrMDA6MDAifQ.BI6l2nvL34N5CpIN3dOVOcCMaQQJ5uJlnVG2Hbyh-6521jojDw3hDDA60mP88mocnoHw0HWsAwus2VcXRHH0Ue2lwda5LRSVG-lnT7hUhwgcerz8_l59Ekgdm8iu9vHHU_kYDFGHLUPc_gemgunVLzwfCCXVxAup_ggoBLoVjOY5mZeMH3li6vgW0PCCmJB6HbkaYEbh_SwgDi9N2LkcmGp4olArU130muEUN6eNPZZp_saKus0bhw24x9hkVDe80f2DRl38lteLvy20Rvy8_ljKAWl1Syh7gXRvvTAEiV7eKMzUdKcrrQgjLfGOxHAPUei2640Z2SgA8Ks2EA-9Fg",
            "total_amount": 1
        }
    }
}
```



### Token usage

You can use received jwt string in order to get access to any protected user information simply by including `context` variable in all requests:


```
{
  "context": {
    "jwt": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJ1c2VyX2lkIjo1ODYsImFjY2VzcyI6ImN1c3RvbWVyIiwiZXhwaXJlIjoiMjAxOS0wNi0xOFQxNTo0NDoxMyswNDowMCJ9.DrGeyReowpizC8UKQRuIK34-CtHXxglkPVOVfyn1FHft0udXKPYqIQJgPJ15jBylYvmcWa13JolICXTkJpuPuQ"
  }
}
```


The alternative way to provide context information is via HTTP request headers. Use:



*   **X-App-Token** to provide JWT string with token
*   **X-App-Currency** to provide currency code
*   **X-App-Language** to provide language code

The headers are being checked only if there is no **context** variable in the request.

There is no permission information provided via `auth` mutation, and the overall scheme can differ only between anonymous and registered user. The auth scheme can be improved to allow for different access levels or available permission, but that will require some modifications.

Each response object has the obligatory `data` property and can include `errors` property:


```
{
  "data": {
    "auth": null
  },
  "errors": [
    {
      "message": "Access denied.",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "auth"
      ]
    }
  ]
}
```


The API access isn’t protected with external application identifiers check - but this can be improved with some custom modification.


## Module-related queries

Some queries in this GraphQL API are being exclusively resolved by various X-Cart addons, e.g. My Wishlist addon, Put an Offer addon. In case when required module is not enabled in the system, the API will return the similar response:


```
{
  "data": {
    "wishlist": null
  },
  "errors": [
    {
      "message": "No module",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "wishlist"
      ]
    }
  ]
}
```


The data object will contain null property and the errors will have the message “No module” - that distinguishes the state when the module is required to be enabled on the store backend and until that the following feature should be disabled in the connected system. 

You can also fetch the list of the installed addons in order to control the available functionality in your app:


```
{
  appData {
    modules {
      name
      enabled
    }
  }
}
```


When the certain addon name (e.g. QSL-MyWishlist) is either non-present in the list or marked as disabled, you can safely disable the corresponding features in your app.


## Examples

Here are the few examples of making GraphQL queries:


### Getting the collection of products filtered by some criteria


```
query ($filters: products_filter_input) {
  collection(from: 0, size: 100, type: products, filters: $filters) {
    count
    objects {
      id
      ... on product {
        product_name
        price
        available
      }
    }
  }
}
```


With the variables: 


```
{
  "filters": {
    "featured": true
  }
}
```



### Searching for products

Use the [Getting the collection of products](#getting-the-collection-of-products-filtered-by-some-criteria) request with the following filters:

With the variables: 


```
{
  "filters": {
    "searchFilter": "substring"
  }
}
```



### Getting user data (requires auth)


```
query {
  user {
    login
    enabled
    first_name
    last_name
    contact_us_url
    account_details_url
    orders_list_url
    address_book_url
    messages_url
  }
}
```


Variables


```
{
  "context": {
    "jwt": "<...large string of jwt token...>"
  }
}
```


The `*_url` fields return a special URL leading to a stripped-down mobile-ready version of the page (suitable for WebView).


### Getting language information


```
query {
  appConfig {
    default_language {
      code
      name
    }
  }
  user {
    language
  }
}
```



### Getting product collections metadata


```
query {
  appData {
    home_page_widgets {
      display_name
      service_name
      enabled
      type
      params {
        filters
      }
    }
  }
}
```


The path `appData.home_page_widgets.params.filters` contains JSON-encoded string of `"filters"`to use in query variables in order to obtain certain collection.


### Getting wishlist data (requires auth)


```
{
  wishlist {
    id
    user_id
    items {
      id
      product_name
    }
    count
  }
}
```



### Register new user


```
mutation($data: UserRegisterInput!) {
  register(data: $data) {
    jwt
    user {
      id
    }
  }
}
```


Variables


```
{
    "data": {
    "email": "test@asdasd.com",
    "password": "123456789",
    "password_conf": "123456789",
    "login": "test@asdasd.com",
    "first_name": "Test",
    "last_name": "Test lastname",
    "company": "Company",
    "url": "https://asdasd.com",
    "tax_number": "123124124"
  }
}
```



### Read user address book (auth protected) 


```
{
    user {
        id
        address_list {
            id
            first_name
            last_name
            country {
                code
                name
            }
            state {
                code
                name
            }
            county # alternative to "state" if "country" obj doesn't have states
            city
            address
            zip
            phone
        }
    }
}
```


 \
Result:


```
{
  "data": {
        "user": {
            "id": "7",
            "address_list": [
                {
                    "id": "357",
                    "first_name": "Vasya",
                    "last_name": "Pupkin",
                    "country": {
                        "code": "US",
                        "name": "United States"
                    },
                    "state": {
                        "code": "CA",
                        "name": "California"
                    },
                    "county": "",
                    "city": "Los Angeles",
                    "address": "1 E asdasd st 222",
                    "zip": "90001",
                    "phone": "+2 232323232"
                }
            ]
        }
    }
}
```



### Update address in address book (auth protected) 


```
mutation($address: address_input!) {
  changeUserAddress(address_id: 7, address: $address) {
      address_list {
          id
          city
      }
  }
}
```


Variables:


```
{
  "address": {
     "city": "Another City"
  }
}
```



### Delete address from address book (auth protected)


```
mutation {
  deleteUserAddress(address_id: 7) {
      address_list {
          id
      }
  }
}
```

## External auth

Version 5.4.1.0 of the GraphQLApi adds the ability to perform an external authorization with the help of the [OAuth2Client addon](https://market.x-cart.com/addons/single-sign-on-with-oauth-2.html). You can retrieve the list of available auth providers via the following query:

```
query {
  appData {
    external_auth_providers {
      display_name
      service_name
      authorize_url
    }
  }
}
```

In order to link the external user account with the X-Cart and receive user GraphQL token, you have to open `authorize_url` in a WebView inside your application and then allow user to proceed with the authorization. Eventually the user will be redirected to the website and you will be able to receive a GraphQL Token via WebView Messaging system. See [WebView implementation](#webview-implementation) for details.

In case you already implemented an auth system inside your app and want to connect it with the X-Cart, you can perform the following mutation:

```
mutation externalAuth ($auth: ExternalAuthInput!) {
  externalAuth (auth: $auth) 
}
```

With this variable format:
```
{
  "auth": {
    "provider": <provider service name> (string),
    "access_token": {
      "access_token": <OAuth2 token string> (required),
      "resource_owner_id": (optional),
      "refresh_token": (optional),
      "expires_in": (optional),
      "expires": (optional),
    }
  }
}
```

This mutation will return a JWT token in case of successful authorization and profile creation inside the backend.

## Checkout flow 

The API can be used to place orders on the store and provide a customer with an ability to perform the payment on the selected payment processor page. Due to complexity, it has to be implemented in a certain way. The general flow is based on the UI, split by 3 major steps: 



*   Specifying shipping\billing addresses, 
*   Choosing the shipping option, 
*   Selecting the payment method. 

The API is ready to place order once **cart** object has its **checkout_ready** property set to **true.** The UI must allow customer to fill all the required data in order to achieve this. You can retrieve the list of the problems, preventing the checkout, from the **errors** array:



*   EMPTY_CART
*   NO_PAYMENT_SELECTED
*   NO_SHIPPING_SELECTED
*   NO_SHIPPING_AVAILABLE
*   NO_SHIPPING_ADDRESS

Cart preparation steps are usually the following:

1. Retrieve latest cart state by using [CheckoutShippingPageQuery](#checkoutshippingpagequery)
2. Specify the address with [ChangeShippingAddress](#changeshippingaddress-mutation-might-change-both-addresses) mutation (this might also change billing address & shipping methods, so we refetch this data too)
    1. You can also change billing address by changing mutation type to “billing”
    2. Alternatively, you can select one of the existing address by using SelectAddress mutation.
3. Choose shipping option with [ChangeShippingMethod](#changeshippingmethod-mutation) mutation
4. Refresh the state with [CheckoutPaymentPageQuery](#checkoutpaymentpagequery)
5. Select payment method with [ChangePaymentMethod](#changepaymentmethod-mutation) mutation
6. Set payment options (if required) with [ChangePaymentFields](#changepaymentfields-mutation) mutation
7. (OPTIONAL) Set customer notes with [ChangeCustomerNotes](#changecustomernotes-mutation) mutation

Once these steps are done and the cart is in `checkout_ready = true` state, you are ready to allow customer to place an order. The payment process is performed inside WebView to allow for various payment providers. See [WebView implementation](#webview-implementation) for details.

### Payment method options 

You can retrieve available payment options from **payment_methods** prop of the **cart** object:


```
  cart {
    id
    user {
      email
    }
    items {
      id
    }
    payment_methods {
      id
      service_name
      payment_name
      details
      fields {
        id
        name
        value
      }
    }
  }
```


In general, all payment_methods won’t require any fields to set up before performing checkout, but you might receive the similar **payment_method** object:


```
        {
          "id": "5",
          "service_name": "Echeck",
          "payment_name": "Check",
          "details": "Check payment",
          "fields": [
            {
              "id": "check_routing_number",
              "name": "ABA routing number",
              "value": ""
            },
            {
              "id": "check_acct_number",
              "name": "Bank Account Number",
              "value": ""
            },
            {
              "id": "check_type",
              "name": "Type of Account",
              "value": ""
            },
            {
              "id": "check_bank_name",
              "name": "Bank name",
              "value": ""
            },
            {
              "id": "check_acct_name",
              "name": "Name of account holder",
              "value": ""
            },
            {
              "id": "check_number",
              "name": "Check number",
              "value": ""
            }
          ]
        },
```


In order to perform successful checkout with such method, you have perform a [ChangePaymentFields mutation](#changepaymentfields-mutation) with a similar request:


```
mutation ($fields: [payment_fields_input]!) {
  changePaymentFields(fields: $fields) {
    payment {
      id
      service_name
      fields {
        id
        name
        value
      }
    }
  }
}
```


With vars:


```
{
  "fields": [{
    "id": "check_routing_number",
    "value": "123456789"
  }],
  "context": {    "jwt":"....jwt...."
  }
}
```


### Coupons feature (requires "Coupons" add-on) 

You can add coupons to the cart by using **addCartCoupon(code)** mutation and remove coupons by using **removeCartCoupon(code)**  mutation. If coupon is valid, it will be applied to the cart and the cart **total** value will be changed. 

The **coupons**property of the cart will contain currently applied coupons.

Also, there is a list of error messages in case coupon code is invalid or cannot be applied for a reason (the message is translated according to the current language):

*   No coupon for **CODE**
*   Sorry, the coupon you entered is invalid. Make sure the coupon code is spelled correctly
*   Sorry, the coupon has expired
*   Sorry, the coupon use limit has been reached
*   You have already used the coupon
*   This coupon cannot be combined with other coupons
*   Sorry, this coupon cannot be combined with the coupon already applied. Revome the previously applied coupon and try again.
*   To use the coupon, your order subtotal must be between **X** and **Y**
*   To use the coupon, your order subtotal must be at least **X**
*   To use the coupon, your order subtotal must not exceed **Y**
*   Sorry, the coupon you entered cannot be applied to the items in your cart
*   Sorry, the coupon you entered is not valid for your membership level. Contact the administrator
*   There is no such a coupon, please check the spelling: **X**

## WebView implementation

To allow for such operations like performing checkout or authorizing user with external auth provider, you have to implement a WebView in a special way to be able to get status updates from the backend and return properly back to your application. The implementation differs for iOS and Android systems.

### iOS WebView implementation

For checkout, use **WKWebView** to open URL stored in `checkout_url` param of the `cart` object. The WebView **must** follow redirects and respond to user clicks. The payment screen will either continue to the success page or proceed to the payment processor. 

In order to return back to the app in case order is placed or errors happened, you should implement Script Messages handler by the name of `status` and react accordingly. Look at [Status message structure](#status-message-structure-and-handling-1) section for details

Very approximate Swift UIViewController implementation might look like this:

```
import UIKit
import WebKit

class MyViewController : UIViewController, WKScriptMessageHandler {

    var userContentController: WKUserContentController?
    
    override func loadView() {
        let url = URL(string: "https://<domain>/admin.php?target=graphql_api_checkout...");
        let config = WKWebViewConfiguration()
        self.userContentController = WKUserContentController()
        config.userContentController = self.userContentController!
        
        let webview = WKWebView(frame: UIView().frame, configuration: config);
        if let unwrapped = url {
            let request = URLRequest(url: unwrapped);
            webview.load(request);
        }
        self.view = webview
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.userContentController!.add(self, name: "status")
    }
    
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {

        if message.name == "status", let messageBody = message.body as? [String: Any] {
            // React somehow on the status
            print(messageBody["last_order_number"]!)
            print(messageBody["messages"]!)
        }
    }
}
```

### Android implementation

For Android app you should use `addJavascriptInteface` feature of the WebView. Look at the guide to implement it: [addJavascriptInterface()](https://developer.android.com/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object,%20java.lang.String)).

GraphQL Api requires you to inject a handler by the name of `statusMessageHandler` with the following interface:

```
class StatusMessageHandler {
  @JavascriptInterface
  public void postMessage(String message) { 
    try {
      JSONObject data = new JSONObject(message);
      // react accordingly to message structure
    } catch (Exception ex) {}
  }
}

webview.getSettings().setJavaScriptEnabled(true);
webView.addJavascriptInterface(new StatusMessageHandler(), "statusMessageHandler");
```

### WebView status message structure and handling

Message is a JSON-like object of the following structure:

```
{
  "last_order_number": "#00001", // string
  "status": "success", // operation status, string,
  "data": [
    // operation-specific format
  ],
  "messages": [
    ["type": "error", 
     "text": "Some text...", 
     "prefix": "Error"]
  ]
}
```

The `status` property indicates the state which can lead to a further actions from the app. 

If `status` is `success`, this means you should stop the WebView and show the  final screen with order confirmation and number from `last_order_number` property to customer.

If `status` is `errors`, you should stop the WebView and return the user on the checkout screen, and display errors from the `messages` array in popup fashion. In other cases `status` property will be an empty string.

If `status` is `auth_success` or `auth_failure`, the user has tried to authorize with the external provider. Read `data` property to get operation details. The `data` format is the following:

```
[
  'success' => true|false, // authorization success, boolean
  'profile_id' => <integer>, // profile id, integer
  'token' => <string>, // JWT string to be used as GraphQL token, string
  'message' => <string>, // status message, string
]
``` 

Notice that you won't have access to the same `cart` object after order is placed.

## Checkout-related queries


#### CheckoutShippingPageQuery


```
query CheckoutShippingPageQuery {
  cart {
    id
    token
    total
    total_amount
    errors
    checkout_ready
    checkout_url
    shipping {
      id
      rate
    }
    payment {
      id
    }
    shipping_address {
      id
      email
      first_name
      last_name
      address
      city
      state {
        code
        name
      }
      country {
        code
        name
      }
      zip
      phone
    }
    billing_address {
      id
      email
      first_name
      last_name
      address
      city
      state {
        code
        name
      }
      country {
        code
        name
      }
      zip
      phone
    }
    shipping_methods {
      id
      shipping_name
      rate
    }
    payment_methods {
      id
    }
  }
}
```



#### ChangeShippingMethod mutation 


```
mutation ChangeShippingMethod($shippingMethodId: ID!) {
   changeShippingMethod (shipping_id: $shippingMethodId) {
     id
     total
     errors
     checkout_ready
     shipping {
       id
       rate
     }
     shipping_methods {
       id
       shipping_name
       rate
     }
   }
 }
```



#### ChangeShippingAddress mutation (might change both addresses) 


```
mutation ChangeShippingAddress($address: address_input!) {
   changeAddress (type: "SHIPPING_TYPE", address: $address) {
     id
     total
     errors
     checkout_ready
     shipping {
       id
       rate
     }
     shipping_address {
       id
       email
       first_name
       last_name
       address
       city
       state {
         code
         name
       }
       country {
         code
         name
       }
       zip
       phone
     }
     billing_address {
       id
       email
       first_name
       last_name
       address
       city
       state {
         code
         name
       }
       country {
         code
         name
       }
       zip
       phone
     }
     shipping_methods {
       id
       shipping_name
       rate
     }
     payment_methods {
       id
     }
   }
 }
```


The **address**variable should contain **country_code**, **state_code**properties as 2-char ISO codes, e.g. “US”, “AZ”. Example:


```
 "address": {
    "email": "bit-bucket@x-cart.com",
    "country_code": "US",
    "state_code": "CA",
    "first_name": "Vasya",
    "last_name": "Petrov",
    "county": "California",
    "city": "Los Angeles",
    "phone": "+18005550505",
    "zipcode": "90001",
  },
```



#### CheckoutPaymentPageQuery


```
query CheckoutPaymentPageQuery {
 cart {
   id
   token
   total
   checkout_ready
   checkout_url
   errors
   payment {
     id
     fields {
       id
       name
     }
   }
   payment_methods {
     id
     service_name
     payment_name
     details
     fields {
       id
       name
       value
     }
   }
 }
}
```



#### ChangePaymentMethod mutation 


```
mutation ChangePaymentMethod($paymentMethodId: ID!) {
   changePaymentMethod (payment_id: $paymentMethodId) {
     id
     payment {
       id
       fields {
         id
         name
         value
       }
     }
   }
 }
```



#### ChangePaymentFields mutation


```
mutation ChangePaymentFields($fields: [payment_fields_input]!) {
   changePaymentFields (fields: $fields) {
     id
     payment {
       id
       fields {
         id
         name
         value
       }
     }
   }
 }
```



#### ChangeCustomerNotes mutation

```
mutation ChangeShippingMethod($notes: String!) {
   changeCustomerNotes (notes: $notes) {
     id
     notes
   }
 }
```
