# AJAX REQUESTS

> This documentation below will describe all the endpoints that are used on the project. For better understanding I have divided the endpoints into several types:

- Universal : used on all pages ( small cart, authorization, registration, etc...)
- Specialized ( checkout, catalog, large cart and ect.).
  In the document below under the symbol ## (Main Title) will be described endpoints that belong to this or that group.

> I also describe below the pattern by which each endpoint was documented

- "###" The common name of the request group (authorization, registration, etc... )
  three characters stand for "###" Number in order ( url )
- ENDPOINT - URL
- EXAMPLE : URL_with example
- TYPE : ENDPOINT TYPE (GET,POST,DELETE ect...)
- RESPONSE`S TYPE (Examples) for different cases (valid, invalid and ect...)
- "####" denotes a direct description of an individual endpoint ( For example, authorization request group, code validation endpoint ).

  > Also, after a short description of each endpoint, a short additional information can be written, detailing particularly important parameters and so on.
  > Symbols "#####" mark the most important cases in each endpoint (e.g. shipping-information endpoint with and without address and so on).

## UNIVERSAL

---

### 1.0 Authorization requests

#### phone check and send sms (/rest/V1a/customer/sms/:phone)

- ENDPOINT - /rest/V1a/customer/sms/:phone
- EXAMPLE - /rest/V1a/customer/sms/380111111111
- TYPE : POST
- RESPONSE ON VALID `{"success":true | Boolean} `
- RESPONSE ON NOT VALID NUMBER `{"success":false | Boolean} `

#### resend the sms code (/rest/V1a/customer/sms/:phone)

> the same request options and params as request above

- ENDPOINT - /rest/V1a/customer/sms/:phone
- EXAMPLE
- TYPE: POST

#### code validation (/rest/V1a/customer/login/:phone/:code)

- ENDPOINT - /rest/V1a/customer/login/:phone/:code (example 380111111111/3123)
- EXAMPLE - /rest/V1a/customer/login/:phone/380111111111/3123
- TYPE: POST
- RESPONSE ON VALID `{"success":true | Boolean} `
- RESPONSE ON NOT VALID NUMBER `{"success":false | Boolean} `

---

### 1.1 Registration requests

#### 1.21 send user data (/rest/V1a/customer/registration/check)

- ENDPOINT - /rest/V1a/customer/registration/check
- TYPE: POST
- REQUEST BODY

```
{
  "phone":"380123123123" | String
  ,"email":"test@gmail.co" | String
}
```

> in case, if customer would be authorized without email, request body seems like:

```
{
  "phone":"380123123123" | String
}
```

- RESPONSE`S TYPES :
  - each of this params signalize that email (OPTIONAL) or phone is used -

```
{
  "email_exists":false | Boolean,
  "phone_exists":true | Boolean
}
```

> if phone and email (OPTIONAL) is not used and customer may continue registration, request changes to :

```
{
  "email_exists":false | Boolean ,
  "phone_exists":false | Boolean
}
```

> but, on sending ONLY phone number :

```
{
  "phone":"380123123123" | String
}
```

> response ALWAYS INCLUDE even if the email parameter is not specified :

```
{
  "email_exists":false | Boolean ,
  "phone_exists":true | Boolean
}
```

#### 1.211 ban on checking register data

> Also after repeatedly entering the wrong code, if the user enters other data for registration, the server response will be as follows

- EXAMPLE OF REQUEST :

```
{
  "phone":"380123124214" | String ,
  "email":"" | String
}
```

- EXAMLPE OF RESPONSE :

```
{
  "banned: 75" | Number
}
```

> ( in this case value in key named "banned" will indicate the time in seconds during which the user will not be able to re-enter the registration data)

#### 1.22 send registration code (/rest/V1a/customer/registration/check_code)

- ENDPOINT - /rest/V1a/customer/registration/check_code
- TYPE: POST
- REQUEST BODY

```
{
  "phone":"380123123125",
  "email":"",
  "name":"тест",
  "lastname":"тест",
  "code":"2412"
}
```

- REQUEST PARAMS :

```
{
  "phone":"380123123125" | String ,
  "email":"" | String ,
  "name":"тест" | String ,
  "lastname":"тест" | String ,
  "code":"2412" | String
}
```

- `"email"` (optional, but always include in request param with value '' if current email not presented in request) -
- RESPONSE ON VALID CODE `{"success":true | Boolean }`
- RESPONSE ON INCORRECT CODE `{"success":false | Boolean }`
- RESPONSE ON EXCEEDING INCORRECT CODE ENTRY ATTEMPTS `{"success":false | Boolean ,"banned":75 | Number }`

### 1.3 choose city popup (/rest/V1/geo/city/?q=:cityName&lang=:locale)

- ENDOINT - /rest/V1/geo/city/?q=:cityName&lang=:locale (uk, ru, en)
- TYPE: GET
- REQUEST EXAMPLE - /rest/V1/geo/city/?q=Київ&lang=uk `{"q": "Київ" , "lang": "uk"}`
- RESPONSE TYPE

```
  [
    {name: "Київка", id: "6522382503", region: "Херсонська область,Голопристанський р-н"},
    {name: "Київець", id: "6522382502", region: "Херсонська область,Голопристанський р-н"}
  ]
```

- RESPONSE ON NOT VALID RESULT

```
[
    {code: 400 , message: "Нічого не знайдено"}
]

```

(the locale of the error message varies from the locale set in the request)

## CHECKOUT

### 2.0 selecting city and requesting info about shipping methods (/rest/V1a/checkout/estimate-shipping-methods/:selected_city_id)

> This request work only on checkout page . This query is executed after successful selection of a city from the list described in above request

- ENDPOINT /rest/V1a/checkout/estimate-shipping-methods/:selected_city_id (id or koaatu)
- REQUEST EXAMPLE : - /rest/V1a/checkout/estimate-shipping-methods/1224881004
- TYPE : GET

#### 2.1 response example (IN THIS CASE ALL SHIPPING METHODS ARE UNAVAILABLE) :

```
[
    {
        "carrier_code": "instore",
        "method_code": "pickup",
        "carrier_title": "Самовивіз з магазину",
        "method_title": "Самовивіз з магазину",
        "amount": 0,
        "base_amount": 0,
        "available": 0,
        "messages": {
            "error": "Нічого не знайдено за даним номером KOATUU 6322284503"
        },
        "additional": []
    },
    {
        "carrier_code": "novaposhta",
        "method_code": "novaposhta_to_warehouse",
        "carrier_title": "Нова пошта",
        "method_title": "Самовивіз із Нової Пошти",
        "amount": 75,
        "base_amount": 0,
        "available": 0,
        "messages": {
            "error": "Нічого не знайдено за даним номером KOATUU 6322284503"
        },
        "additional": {
            "warehouse": {
                "amount": 0,
                "available": true
            },
            "postomat": {
                "amount": 0,
                "available": true
            },
            "courier": {
                "available": true
            }
        }
    }
]
```

#### response params :

- "carrier_code" - type of delivery method
- "carrier_title" - name of delivery type in current locale
- "amount" - price of delivery shipping
- "available" - signalize that this method is available for this location , 0 - not available, 1 - available
- "additional" : {} - an array with additional delivery methods, they include other methods, as warehouse, postomat, courier, and each of this methods has an keys, that signalize is current method available and value of contained entries ("amount" & "available")

#### 2.2 response example (ALL AVAILABLE SHIPPING METHODS) :

```
[
    {
        "carrier_code": "instore",
        "method_code": "pickup",
        "carrier_title": "Самовивіз з магазину",
        "method_title": "Самовивіз з магазину",
        "amount": 0,
        "base_amount": 0,
        "available": 1,
        "messages": [],
        "additional": []
    },
    {
        "carrier_code": "novaposhta",
        "method_code": "novaposhta_to_warehouse",
        "carrier_title": "Нова пошта",
        "method_title": "Самовивіз із Нової Пошти",
        "amount": 75,
        "base_amount": 0,
        "available": 1,
        "messages": [],
        "additional": {
            "warehouse": {
                "amount": 98,
                "available": true
            },
            "postomat": {
                "amount": 1683,
                "available": true
            },
            "courier": {
                "available": true
            }
        }
    }
]
```

### 2.1 get available shipping addresses for instore (/:locale/rest/V1a/inventory/in-store-pickup/pickup-locations/:selected_city_id)

- ENDPOINT - /:locale/rest/V1a/inventory/in-store-pickup/pickup-locations/:selected_city_id
- TYPE: GET
- REQUEST EXAMPLE - /uk/rest/V1a/inventory/in-store-pickup/pickup-locations/5110100000
- RESPONSE EXAMPLE

```
[
    {
        "source_code": "БАККАРА",
        "name": "БАККАРА",
        "latitude": null,
        "longitude": null,
        "index_coatsu1": "5110100000",
        "frontend": {
            "name": "вул. Академіка Філатова, 5/2",
            "description": ""
        }
    },
    {
        "source_code": "ВОЛНА",
        "name": "ВОЛНА",
        "latitude": null,
        "longitude": null,
        "index_coatsu1": "5110100000",
        "frontend": {
            "name": "вул. Семена Палія, 127/3 - ТЦ \"ВОЛНА\"",
            "description": ""
        }
    },
    {
        "source_code": "ВУЗОВСКИЙ",
        "name": "ВУЗОВСКИЙ",
        "latitude": null,
        "longitude": null,
        "index_coatsu1": "5110100000",
        "frontend": {
            "name": "вул. Олександра Невського, 57",
            "description": ""
        }
    }
]
```

### 2.2 get available shipping addresses ( WAREHOUSE /rest/:locale/V1a/novaposhta/warehouses/)

- ENDPOINT - /rest/:locale/V1a/novaposhta/warehouses/
- TYPE: GET
- REQUEST EXAMPLE `/rest/ru/V1a/novaposhta/warehouses/5110100000?searchCriteria%5Bfilter_groups%5D%5B0%5D%5Bfilters%5D%5B0%5D%5Bfield%5D=type&searchCriteria%5Bfilter_groups%5D%5B0%5D%5Bfilters%5D%5B0%5D%5Bvalue%5D=warehouse&searchCriteria%5BcurrentPage%5D=1&searchCriteria%5BpageSize%5D=50`

#### 2.21 example without customer input

- REQUEST BODY

```

{
"searchCriteria[filter_groups][0][filters][0][field]": "type"
"searchCriteria[filter_groups][0][filters][0][value]": "warehouse"
"searchCriteria[currentPage]": "1"
"searchCriteria[pageSize]": "50"
}

```

```
{
    "items": [
        {
            "value": 4753,
            "label": "\tВідділення №154 (до 10 кг): вул. Академіка Філатова, 1 (маг.\"Сільпо\")",
            "ref": "1ec37a62-ed6f-11ec-9eb1-d4f5ef0df2b8",
            "type": "warehouse"
        },
        {
            "value": 19885,
            "label": "Відділення №10 (до 30 кг на одне місце): вул.Генерала Бочарова 28",
            "ref": "16922802-e1c2-11e3-8c4a-0050568002cf",
            "type": "warehouse"
        },
        {
            "value": 14750,
            "label": "Відділення №101 (до 10 кг): с. Лиманка, \"Радужний\" масив ж/масиву \"Ульянівка\", 23, прим. 7",
            "ref": "89fc60e1-f973-11e4-8a92-005056887b8d",
            "type": "warehouse"
        },
        {
            "value": 16284,
            "label": "Відділення №102 (до 10 кг): смт. Авангард, вул. Європейська, 9, секція 1, прим. 1В",
            "ref": "98793f4e-f973-11e4-8a92-005056887b8d",
            "type": "warehouse"
        },
    ],
    "search_criteria": {
        "filter_groups": [
            {
                "filters": [
                    {
                        "field": "type",
                        "value": "warehouse",
                        "condition_type": "eq"
                    }
                ]
            }
        ],
        "page_size": 50,
        "current_page": 1
    },
    "total_count": 147
}
```

#### 2.23 example with customer input

- REQUEST

```
{
  "searchCriteria[filter_groups][0][filters][0][field]": "type"
  "searchCriteria[filter_groups][0][filters][0][value]": "warehouse"
  "searchCriteria[currentPage]": 1
  "searchCriteria[pageSize]": 50
  "searchCriteria[filter_groups][1][filters][0][field]": "search"
  "searchCriteria[filter_groups][1][filters][0][value]": "Хар"
}
```

- RESPONSE

```
{
    "items": [
        {
            "value": 7366,
            "label": "Відділення №23 (до 30 кг): с. Крижанівка, вул. Сахарова Академіка 3к",
            "ref": "5a39e590-e1c2-11e3-8c4a-0050568002cf",
            "type": "warehouse"
        }
    ],
    "search_criteria": {
        "filter_groups": [
            {
                "filters": [
                    {
                        "field": "type",
                        "value": "warehouse",
                        "condition_type": "eq"
                    }
                ]
            },
            {
                "filters": [
                    {
                        "field": "search",
                        "value": "Хар",
                        "condition_type": "eq"
                    }
                ]
            }
        ],
        "page_size": 50,
        "current_page": 1
    },
    "total_count": 1
}
```

### 2.3 get available shipping addresses ( POSTOMAT /rest/:locale/V1a/novaposhta/warehouses/)

- ENDPOINT - /rest/:locale/V1a/novaposhta/warehouses/ ( works like request for warehouses )
- TYPE: GET
- REQUEST EXAMPLES

#### 2.31 example without customer input

```

{
"searchCriteria[filter_groups][0][filters][0][field]": "type"
"searchCriteria[filter_groups][0][filters][0][value]": "warehouse"
"searchCriteria[currentPage]": "1"
"searchCriteria[pageSize]": "50"
}

```

- RESPONSE TYPE :

```
{
    "items": [
        {
            "value": 17529,
            "label": "Поштомат \"Нова Пошта\" №1347: вул. Проценка, 50, корпус 1, під'їзд №2 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)",
            "ref": "8e94118c-8cbb-11eb-9797-b8830365bd14",
            "type": "postomat"
        },
        {
            "value": 22416,
            "label": "Поштомат \"Нова Пошта\" №1379: вул. Шота Руставелі, 9, під’їзд №1 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)",
            "ref": "8e9d4568-8cbb-11eb-9797-b8830365bd14",
            "type": "postomat"
        },
        {
            "value": 1771,
            "label": "Поштомат \"Нова Пошта\" №1380: вул. Шота Руставелі, 9, під’їзд №2 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)",
            "ref": "b337d064-8cbb-11eb-9797-b8830365bd14",
            "type": "postomat"
        },
        {
            "value": 18061,
            "label": "Поштомат \"Нова Пошта\" №1381: вул. Шота Руставелі, 9, під’їзд №3 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)",
            "ref": "b33b5ea4-8cbb-11eb-9797-b8830365bd14",
            "type": "postomat"
        },
        {
            "value": 14086,
            "label": "Поштомат \"Нова Пошта\" №1382: вул. Шота Руставелі, 9, під’їзд №4 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)",
            "ref": "da354cb0-8cbb-11eb-9797-b8830365bd14",
            "type": "postomat"
        },
    ],
    "search_criteria": {
        "filter_groups": [
            {
                "filters": [
                    {
                        "field": "type",
                        "value": "postomat",
                        "condition_type": "eq"
                    }
                ]
            }
        ],
        "page_size": 50,
        "current_page": 1
    },
    "total_count": 715
}
```

#### 2.32 example with customer input

- REQUEST :

```
{
    "searchCriteria[filter_groups][0][filters][0][field]": "type"
    "searchCriteria[filter_groups][0][filters][0][value]": "postomat"
    "searchCriteria[currentPage]": 1
    "searchCriteria[pageSize]": 50
    "searchCriteria[filter_groups][1][filters][0][field]": "search"
    "searchCriteria[filter_groups][1][filters][0][value]": "Хар"
}

```

- RESPONSE :

```
{
    "items": [
        {
            "value": 492,
            "label": "Поштомат \"Нова Пошта\" №1466: вул. Сахарова, 18, під’їзд №1 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)",
            "ref": "d80fe94f-9057-11eb-85ca-b8830365bd14",
            "type": "postomat"
        },
        {
            "value": 10558,
            "label": "Поштомат \"Нова Пошта\" №1564: вул. Академіка Сахарова, 24, під'їзд №1 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)",
            "ref": "9941ae3d-92bc-11eb-8df8-b8830365bd14",
            "type": "postomat"
        },
        {
            "value": 13162,
            "label": "Поштомат \"Нова Пошта\" №1565: вул. Академіка Сахарова, 24, під'їзд №2 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)",
            "ref": "4d814b79-92bd-11eb-8df8-b8830365bd14",
            "type": "postomat"
        },
    ],
    "search_criteria": {
        "filter_groups": [
            {
                "filters": [
                    {
                        "field": "type",
                        "value": "postomat",
                        "condition_type": "eq"
                    }
                ]
            },
            {
                "filters": [
                    {
                        "field": "search",
                        "value": "Хар",
                        "condition_type": "eq"
                    }
                ]
            }
        ],
        "page_size": 50,
        "current_page": 1
    },
    "total_count": 19
}
```

### 2.4 get available shipping street ( COURIER /rest/uk/V1a/novaposhta/streets/:selected_city_id)

> Works like two previous requests, but request include customer input value in `searchCriteria[filter_groups][0][filters][0][value]: "Назва_вулиці"`

- ENDPOINT - /rest/uk/V1a/novaposhta/streets/:selected_city_id
- TYPE: POST
- REQUEST EXAMPLE :

```

{
    "searchCriteria[filter_groups][0][filters][0][field]": "search"
    "searchCriteria[filter_groups][0][filters][0][value]": "Харк"
    "searchCriteria[currentPage]": 1
    "searchCriteria[pageSize]": 50
}

```

- RESPONSE EXAMPLE :

```
{
    "items": [
        {
            "value": 187193,
            "label": "Сахарова Академіка",
            "type": "вул."
        },
        {
            "value": 187456,
            "label": "Харківська",
            "type": "вул."
        }
    ],
    "search_criteria": {
        "filter_groups": [
            {
                "filters": [
                    {
                        "field": "search",
                        "value": "Хар",
                        "condition_type": "eq"
                    }
                ]
            }
        ],
        "page_size": 50,
        "current_page": 1
    },
    "total_count": 2
}
```

- RESPONSE WITHOUT RESULTS :

```
{
    "items": [],
    "search_criteria": {
        "filter_groups": [
            {
                "filters": [
                    {
                        "field": "search",
                        "value": "Хара",
                        "condition_type": "eq"
                    }
                ]
            }
        ],
        "page_size": 50,
        "current_page": 1
    },
    "total_count": 0
}

```

### 2.5 set shipping information about order (/:locale/rest/V1a/carts/mine/shipping-information)

- ENDPOINT - /:locale/rest/V1a/carts/mine/shipping-information
- TYPE : POST
- REQUEST EXAMPLES

> "warehouse_ref" - optional param, it can be omitted and the request will be executed successfully for all cases described below (instore, warehouse, post office or address delivery).
> also the value in `"street" []` param optional, if we just want to calculate the shipping cost, we use "street": ["withoutData"]

#### 2.51 shipping ( INSTORE ) (/:locale/rest/V1a/carts/mine/shipping-information)

##### request example with pickup_location_code

> the value of this attribute is taken after selecting a store from the
> drop-down list at the address `/:locale/rest/V1a/inventory/in-store-pickup/pickup-locations/`

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "pickup_location_code": "ВОЛНА"
            }
        },
        "shipping_method_code": "pickup",
        "shipping_carrier_code": "instore"
    }
}
```

- RESPONSE :

```
{
    "payment_methods": [
        {
            "code": "cashondelivery",
            "title": "При отриманні"
        },
        {
            "code": "antoshka_liqpay",
            "title": "LiqPay"
        }
    ],
    "totals": {
        "grand_total": 101,
        "base_grand_total": 101,
        "subtotal": 101,
        "base_subtotal": 101,
        "discount_amount": 0,
        "base_discount_amount": 0,
        "subtotal_with_discount": 101,
        "base_subtotal_with_discount": 101,
        "shipping_amount": 0,
        "base_shipping_amount": 0,
        "shipping_discount_amount": 0,
        "base_shipping_discount_amount": 0,
        "tax_amount": 0,
        "base_tax_amount": 0,
        "weee_tax_applied_amount": null,
        "shipping_tax_amount": 0,
        "base_shipping_tax_amount": 0,
        "subtotal_incl_tax": 101,
        "shipping_incl_tax": 0,
        "base_shipping_incl_tax": 0,
        "base_currency_code": "UAH",
        "quote_currency_code": "UAH",
        "items_qty": 1,
        "items": [
            {
                "item_id": 576,
                "price": 101,
                "base_price": 101,
                "qty": 1,
                "row_total": 101,
                "base_row_total": 101,
                "row_total_with_discount": 0,
                "tax_amount": 0,
                "base_tax_amount": 0,
                "tax_percent": 0,
                "discount_amount": 0,
                "base_discount_amount": 0,
                "discount_percent": 0,
                "price_incl_tax": 101,
                "base_price_incl_tax": 101,
                "row_total_incl_tax": 101,
                "base_row_total_incl_tax": 101,
                "options": "[]",
                "weee_tax_applied_amount": null,
                "weee_tax_applied": null,
                "name": "Пов'язка BluKids Retro Baby"
            }
        ],
        "total_segments": [
            {
                "code": "subtotal",
                "title": "Subtotal",
                "value": 101
            },
            {
                "code": "giftwrapping",
                "title": "Gift Wrapping",
                "value": null,
                "extension_attributes": {
                    "gw_item_ids": [],
                    "gw_price": "0.0000",
                    "gw_base_price": "0.0000",
                    "gw_items_price": "0.0000",
                    "gw_items_base_price": "0.0000",
                    "gw_card_price": "0.0000",
                    "gw_card_base_price": "0.0000"
                }
            },
            {
                "code": "shipping",
                "title": "Shipping & Handling (Самовивіз з магазину - вул. Академіка Філатова, 5/2)",
                "value": 0
            },
            {
                "code": "tax",
                "title": "Tax",
                "value": 0,
                "extension_attributes": {
                    "tax_grandtotal_details": []
                }
            },
            {
                "code": "grand_total",
                "title": "Grand Total",
                "value": 101,
                "area": "footer"
            }
        ],
        "extension_attributes": {
            "reward_points_balance": 0,
            "reward_currency_amount": 0,
            "base_reward_currency_amount": 0
        }
    }
}
```

#### 2.52 shipping ( WAREHOUSE ) (/:locale/rest/V1a/carts/mine/shipping-information)

##### request example with "street": value and "warehouse_ref" :

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000",
                "warehouse_ref": "16922802-e1c2-11e3-8c4a-0050568002cf"
            },
            "street": [
                "Відділення №10 (до 30 кг на одне місце): вул.Генерала Бочарова 28"
            ]
        },
        "shipping_method_code": "novaposhta_to_warehouse",
        "shipping_carrier_code": "novaposhta"
    }
}

```

- RESPONSE :

```
{
    "payment_methods": [
        {
            "code": "cashondelivery",
            "title": "При отриманні"
        },
        {
            "code": "antoshka_liqpay",
            "title": "LiqPay"
        }
    ],
    "totals": {
        "grand_total": 176,
        "base_grand_total": 176,
        "subtotal": 101,
        "base_subtotal": 101,
        "discount_amount": 0,
        "base_discount_amount": 0,
        "subtotal_with_discount": 101,
        "base_subtotal_with_discount": 101,
        "shipping_amount": 75,
        "base_shipping_amount": 75,
        "shipping_discount_amount": 0,
        "base_shipping_discount_amount": 0,
        "tax_amount": 0,
        "base_tax_amount": 0,
        "weee_tax_applied_amount": null,
        "shipping_tax_amount": 0,
        "base_shipping_tax_amount": 0,
        "subtotal_incl_tax": 101,
        "shipping_incl_tax": 75,
        "base_shipping_incl_tax": 75,
        "base_currency_code": "UAH",
        "quote_currency_code": "UAH",
        "items_qty": 1,
        "items": [
            {
                "item_id": 537,
                "price": 101,
                "base_price": 101,
                "qty": 1,
                "row_total": 101,
                "base_row_total": 101,
                "row_total_with_discount": 0,
                "tax_amount": 0,
                "base_tax_amount": 0,
                "tax_percent": 0,
                "discount_amount": 0,
                "base_discount_amount": 0,
                "discount_percent": 0,
                "price_incl_tax": 101,
                "base_price_incl_tax": 101,
                "row_total_incl_tax": 101,
                "base_row_total_incl_tax": 101,
                "options": "[]",
                "weee_tax_applied_amount": null,
                "weee_tax_applied": null,
                "name": "Пов'язка BluKids Retro Baby"
            }
        ],
        "total_segments": [
            {
                "code": "subtotal",
                "title": "Subtotal",
                "value": 101
            },
            {
                "code": "giftwrapping",
                "title": "Gift Wrapping",
                "value": null,
                "extension_attributes": {
                    "gw_item_ids": [],
                    "gw_price": "0.0000",
                    "gw_base_price": "0.0000",
                    "gw_items_price": "0.0000",
                    "gw_items_base_price": "0.0000",
                    "gw_card_price": "0.0000",
                    "gw_card_base_price": "0.0000"
                }
            },
            {
                "code": "shipping",
                "title": "Shipping & Handling (Нова пошта - Самовивіз із Нової Пошти)",
                "value": 75
            },
            {
                "code": "tax",
                "title": "Tax",
                "value": 0,
                "extension_attributes": {
                    "tax_grandtotal_details": []
                }
            },
            {
                "code": "grand_total",
                "title": "Grand Total",
                "value": 176,
                "area": "footer"
            }
        ],
        "extension_attributes": {
            "reward_points_balance": 0,
            "reward_currency_amount": 0,
            "base_reward_currency_amount": 0
        }
    }
}
```

##### without "street": [] and "warehouse_ref" :

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000"
            },
            "street": [
                "withoutData"
            ]
        },
        "shipping_method_code": "novaposhta_to_warehouse",
        "shipping_carrier_code": "novaposhta"
    }
}

```

#### 2.53 shipping ( POSTOMAT ) (/:locale/rest/V1a/carts/mine/shipping-information)

##### request example with "street": value and "warehouse_ref" :

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000",
                "warehouse_ref": "8e9d4568-8cbb-11eb-9797-b8830365bd14"
            },
            "street": [
                "Поштомат \"Нова Пошта\" №1379: вул. Шота Руставелі, 9, під’їзд №1 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)"
            ]
        },
        "shipping_method_code": "novaposhta_to_warehouse",
        "shipping_carrier_code": "novaposhta"
    }
}
```

##### request example without "street": value and "warehouse_ref" :

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000"
            },
            "street": [
                "withoutData"
            ]
        },
        "shipping_method_code": "novaposhta_to_warehouse",
        "shipping_carrier_code": "novaposhta"
    }
}
```

- RESPONSE :

```
{
    "payment_methods": [
        {
            "code": "cashondelivery",
            "title": "При отриманні"
        },
        {
            "code": "antoshka_liqpay",
            "title": "LiqPay"
        }
    ],
    "totals": {
        "grand_total": 176,
        "base_grand_total": 176,
        "subtotal": 101,
        "base_subtotal": 101,
        "discount_amount": 0,
        "base_discount_amount": 0,
        "subtotal_with_discount": 101,
        "base_subtotal_with_discount": 101,
        "shipping_amount": 75,
        "base_shipping_amount": 75,
        "shipping_discount_amount": 0,
        "base_shipping_discount_amount": 0,
        "tax_amount": 0,
        "base_tax_amount": 0,
        "weee_tax_applied_amount": null,
        "shipping_tax_amount": 0,
        "base_shipping_tax_amount": 0,
        "subtotal_incl_tax": 101,
        "shipping_incl_tax": 75,
        "base_shipping_incl_tax": 75,
        "base_currency_code": "UAH",
        "quote_currency_code": "UAH",
        "items_qty": 1,
        "items": [
            {
                "item_id": 537,
                "price": 101,
                "base_price": 101,
                "qty": 1,
                "row_total": 101,
                "base_row_total": 101,
                "row_total_with_discount": 0,
                "tax_amount": 0,
                "base_tax_amount": 0,
                "tax_percent": 0,
                "discount_amount": 0,
                "base_discount_amount": 0,
                "discount_percent": 0,
                "price_incl_tax": 101,
                "base_price_incl_tax": 101,
                "row_total_incl_tax": 101,
                "base_row_total_incl_tax": 101,
                "options": "[]",
                "weee_tax_applied_amount": null,
                "weee_tax_applied": null,
                "name": "Пов'язка BluKids Retro Baby"
            }
        ],
        "total_segments": [
            {
                "code": "subtotal",
                "title": "Subtotal",
                "value": 101
            },
            {
                "code": "giftwrapping",
                "title": "Gift Wrapping",
                "value": null,
                "extension_attributes": {
                    "gw_item_ids": [],
                    "gw_price": "0.0000",
                    "gw_base_price": "0.0000",
                    "gw_items_price": "0.0000",
                    "gw_items_base_price": "0.0000",
                    "gw_card_price": "0.0000",
                    "gw_card_base_price": "0.0000"
                }
            },
            {
                "code": "shipping",
                "title": "Shipping & Handling (Нова пошта - Самовивіз із Нової Пошти)",
                "value": 75
            },
            {
                "code": "tax",
                "title": "Tax",
                "value": 0,
                "extension_attributes": {
                    "tax_grandtotal_details": []
                }
            },
            {
                "code": "grand_total",
                "title": "Grand Total",
                "value": 176,
                "area": "footer"
            }
        ],
        "extension_attributes": {
            "reward_points_balance": 0,
            "reward_currency_amount": 0,
            "base_reward_currency_amount": 0
        }
    }
}
```

#### 2.54 shipping ( COURIER ) (/:locale/rest/V1a/carts/mine/shipping-information)

##### request example with "street: value"

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000"
            },
            "street": [
                "просп. Шевченка",
                "123",
                "42"
            ]
        },
        "shipping_method_code": "novaposhta_to_door",
        "shipping_carrier_code": "novaposhta"
    }
}

```

##### without "street" : []

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000"
            },
            "street": [
                "withoutData"
            ]
        },
        "shipping_method_code": "novaposhta_to_door",
        "shipping_carrier_code": "novaposhta"
    }
}

```

- RESPONSE :

```
{
    "payment_methods": [
        {
            "code": "cashondelivery",
            "title": "При отриманні"
        },
        {
            "code": "antoshka_liqpay",
            "title": "LiqPay"
        }
    ],
    "totals": {
        "grand_total": 301,
        "base_grand_total": 301,
        "subtotal": 101,
        "base_subtotal": 101,
        "discount_amount": 0,
        "base_discount_amount": 0,
        "subtotal_with_discount": 101,
        "base_subtotal_with_discount": 101,
        "shipping_amount": 200,
        "base_shipping_amount": 200,
        "shipping_discount_amount": 0,
        "base_shipping_discount_amount": 0,
        "tax_amount": 0,
        "base_tax_amount": 0,
        "weee_tax_applied_amount": null,
        "shipping_tax_amount": 0,
        "base_shipping_tax_amount": 0,
        "subtotal_incl_tax": 101,
        "shipping_incl_tax": 200,
        "base_shipping_incl_tax": 200,
        "base_currency_code": "UAH",
        "quote_currency_code": "UAH",
        "items_qty": 1,
        "items": [
            {
                "item_id": 537,
                "price": 101,
                "base_price": 101,
                "qty": 1,
                "row_total": 101,
                "base_row_total": 101,
                "row_total_with_discount": 0,
                "tax_amount": 0,
                "base_tax_amount": 0,
                "tax_percent": 0,
                "discount_amount": 0,
                "base_discount_amount": 0,
                "discount_percent": 0,
                "price_incl_tax": 101,
                "base_price_incl_tax": 101,
                "row_total_incl_tax": 101,
                "base_row_total_incl_tax": 101,
                "options": "[]",
                "weee_tax_applied_amount": null,
                "weee_tax_applied": null,
                "name": "Пов'язка BluKids Retro Baby"
            }
        ],
        "total_segments": [
            {
                "code": "subtotal",
                "title": "Subtotal",
                "value": 101
            },
            {
                "code": "giftwrapping",
                "title": "Gift Wrapping",
                "value": null,
                "extension_attributes": {
                    "gw_item_ids": [],
                    "gw_price": "0.0000",
                    "gw_base_price": "0.0000",
                    "gw_items_price": "0.0000",
                    "gw_items_base_price": "0.0000",
                    "gw_card_price": "0.0000",
                    "gw_card_base_price": "0.0000"
                }
            },
            {
                "code": "shipping",
                "title": "Shipping & Handling (Нова пошта - Доставка кур'єром до дверей)",
                "value": 200
            },
            {
                "code": "tax",
                "title": "Tax",
                "value": 0,
                "extension_attributes": {
                    "tax_grandtotal_details": []
                }
            },
            {
                "code": "grand_total",
                "title": "Grand Total",
                "value": 301,
                "area": "footer"
            }
        ],
        "extension_attributes": {
            "reward_points_balance": 0,
            "reward_currency_amount": 0,
            "base_reward_currency_amount": 0
        }
    }
}
```

### 2.6 create order (/:locale/rest/V1a/checkout/create-order)

- ENDPOINT - /:locale/rest/V1a/checkout/create-order
- TYPE : POST
- IN CREATE ORDER PARAMS "warehouse_ref" and "street:" [] are REQURIED
- REQUEST EXAMPLES

#### 2.61 request for create order ( INSTORE )

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "pickup_location_code": "БАККАРА"
            }
        },
        "shipping_method_code": "pickup",
        "shipping_carrier_code": "instore"
    }
}
```

#### 2.61 response for create order ( INSTORE )

```
{
    "order_id": 13,
    "increment_id": "500000012",
    "redirect_url": "https://m2amida.antoshka.ua/uk/checkout/onepage/success/"
}
```

#### 2.62 request for create order ( WAREHOUSE )

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000",
                "warehouse_ref": "1ec37a62-ed6f-11ec-9eb1-d4f5ef0df2b8"
            },
            "street": [
                "\tВідділення №154 (до 10 кг): вул. Академіка Філатова, 1 (маг.\"Сільпо\")"
            ]
        },
        "shipping_method_code": "novaposhta_to_warehouse",
        "shipping_carrier_code": "novaposhta"
    }
}
```

#### 2.62 response for create order ( WAREHOUSE )

```
{
    "order_id": 14,
    "increment_id": "500000013",
    "redirect_url": "https://m2amida.antoshka.ua/uk/checkout/onepage/success/"
}
```

#### 2.63 request for create order ( POSTOMAT )

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000",
                "warehouse_ref": "8e9d4568-8cbb-11eb-9797-b8830365bd14"
            },
            "street": [
                "Поштомат \"Нова Пошта\" №1379: вул. Шота Руставелі, 9, під’їзд №1 (ТІЛЬКИ ДЛЯ МЕШКАНЦІВ)"
            ]
        },
        "shipping_method_code": "novaposhta_to_warehouse",
        "shipping_carrier_code": "novaposhta"
    }
}
```

#### 2.63 response for create order ( POSTOMAT )

```
{
    "order_id": 15,
    "increment_id": "500000014",
    "redirect_url": "https://m2amida.antoshka.ua/uk/checkout/onepage/success/"
}
```

#### 2.64 request for create order ( COURIER )

```
{
    "addressInformation": {
        "shipping_address": {
            "firstname": "Шапочка",
            "lastname": "Червона",
            "extension_attributes": {
                "koatuu": "5110100000"
            },
            "street": [
                "вул. Пушкінська",
                "Номер_Будинку",
                "Квартира"
            ]
        },
        "shipping_method_code": "novaposhta_to_door",
        "shipping_carrier_code": "novaposhta"
    }
}
```

#### 2.64 response for create order ( COURIER )

```
{
    "order_id": 16,
    "increment_id": "500000015",
    "redirect_url": "https://m2amida.antoshka.ua/uk/checkout/onepage/success/"
}
```

## CART

### 3.0 get cart data (/:locale/rest/V1a/customer/cart/:quoteId)

- ENDPOINT : /:locale/rest/V1a/customer/cart/:quoteId
- EXAMPLE : /uk/rest/V1a/customer/cart/196
- TYPE : POST
- REQUEST PARAMS : `{"quoteId": 123 | Number}`

  > The most important parameters: "status" - can have the value "fulfilled" if there are goods in the cart and "empty" if the cart is empty. It is also necessary to pay attention to "authorized" - whether the user is authorized, and if so, then in requests for increment, decrement and deletion of goods will be present also Bearer Token, which can be found in the cookie by the name "token".

- TOKEN EXAMPLE : `"token" : "eyJraWQiOiIxIiwiYWxnIjoiSFMyNTYifQ.eyJ1aWQiOjEsInV0eXBpZCI6MywiaWF0IjoxNzEwNDkzOTg5LCJleHAiOjE3MTA0OTc1ODl9.eubIKzcqHVb4B18_wc1Z7JoehQGw4xQ7isLUPRbJnM4"`

#### response for fulfilled cart (ALL ITEMS AVAILABLE)

```
{
    "status": "fulfilled",
    "totalPrice": 101,
    "authorized": true,
    "items_count": 1,
    "products_count": 1,
    "customer_id": 1,
    "quote_id": 196,
    "items": [
        {
            "product_id": 1,
            "product_name": "Пов'язка BluKids Retro Baby",
            "product_sku": "4779679",
            "product_url": "https://m2amida.antoshka.ua/uk/pov-jazka-blukids-retro-baby.html",
            "product_price": 101,
            "qty": 1,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/7/4779679_01.jpg",
            "item_id": 576,
            "in_stock": true
        }
    ]
}
```

#### response for empty cart

```
{
    "status": "empty" | String
}
```

#### response for items with out of stock

```
{
    "status": "fulfilled",
    "totalPrice": 157,
    "authorized": true,
    "items_count": 1,
    "products_count": 1,
    "customer_id": 1,
    "quote_id": 196,
    "items": [
        {
            "product_id": 141,
            "product_name": "Test Test Test",
            "product_sku": "Test Test Test",
            "product_url": "https://m2amida.antoshka.ua/uk/test-test-test.html",
            "product_price": 157,
            "qty": 1,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/7/4779679_01_1_1.jpg",
            "item_id": 577,
            "in_stock": false
        }
    ]
}
```

### 3.1 increment quantity request (/:locale/rest/V1a/customer/changeQty/)

- ENDPOINT - /:locale/rest/V1a/customer/changeQty/
- EXAMPLE - /uk/rest/V1a/customer/changeQty/
- TYPE: POST
- REQUEST BODY
- IF CUSTOMER AUTHORIZED REQUEST HEADER MUST INCLUDE Bearer Token, example

```
Authorization:
Bearer eyJraWQiOiIxIiwiYWxnIjoiSFMyNTYifQ.eyJ1aWQiOjEsInV0eXBpZCI6MywiaWF0IjoxNzEwNDkzOTg5LCJleHAiOjE3MTA0OTc1ODl9.eubIKzcqHVb4B18_wc1Z7JoehQGw4xQ7isLUPRbJnM4
```

```
{
    "quoteNum": 196, | Number
    "itemId": 580, | Number
    "qty": "+1" | String
}
```

#### response type (WITH SUCCESS INCREMENT)

```
{
    "status": "fulfilled",
    "totalPrice": 478,
    "authorized": true,
    "items_count": 3,
    "products_count": 4,
    "customer_id": 1,
    "quote_id": 196,
    "items": [
        {
            "product_id": 1,
            "product_name": "Пов'язка BluKids Retro Baby",
            "product_sku": "4779679",
            "product_url": "https://m2amida.antoshka.ua/uk/pov-jazka-blukids-retro-baby.html",
            "product_price": 101,
            "qty": 1,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/7/4779679_01.jpg",
            "item_id": 578,
            "in_stock": true
        },
        {
            "product_id": 3,
            "product_name": "Набір аксесуарів Coralico Ніжна квітка, 3 шт",
            "product_sku": "4696127",
            "product_url": "https://m2amida.antoshka.ua/uk/nabir-aksesuariv-coralico-nizhna-kvitka-3-sht.html",
            "product_price": 139,
            "qty": 1,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/6/4696127.jpg",
            "item_id": 579,
            "in_stock": true
        },
        {
            "product_id": 2,
            "product_name": "Набір аксесуарів для волосся Coralico Amour 4 шт",
            "product_sku": "4683549",
            "product_url": "https://m2amida.antoshka.ua/uk/nabir-aksesuariv-dlja-volossja-coralico-amour-4-sht.html",
            "product_price": 119,
            "qty": 2,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/6/4683549_04.jpg",
            "item_id": 580,
            "in_stock": true
        }
    ]
}
```

#### response type with error in incrementation

```
{
    "status": "fulfilled",
    "totalPrice": 597,
    "authorized": true,
    "items_count": 3,
    "products_count": 5,
    "customer_id": 1,
    "quote_id": 196,
    "items": [
        {
            "product_id": 1,
            "product_name": "Пов'язка BluKids Retro Baby",
            "product_sku": "4779679",
            "product_url": "https://m2amida.antoshka.ua/uk/pov-jazka-blukids-retro-baby.html",
            "product_price": 101,
            "qty": 1,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/7/4779679_01.jpg",
            "item_id": 578,
            "in_stock": true
        },
        {
            "product_id": 3,
            "product_name": "Набір аксесуарів Coralico Ніжна квітка, 3 шт",
            "product_sku": "4696127",
            "product_url": "https://m2amida.antoshka.ua/uk/nabir-aksesuariv-coralico-nizhna-kvitka-3-sht.html",
            "product_price": 139,
            "qty": 1,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/6/4696127.jpg",
            "item_id": 579,
            "in_stock": true
        },
        {
            "product_id": 2,
            "product_name": "Набір аксесуарів для волосся Coralico Amour 4 шт",
            "product_sku": "4683549",
            "product_url": "https://m2amida.antoshka.ua/uk/nabir-aksesuariv-dlja-volossja-coralico-amour-4-sht.html",
            "product_price": 119,
            "qty": 3,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/6/4683549_04.jpg",
            "item_id": 580,
            "in_stock": true
        }
    ],
    "max_item_qty": 3,
    "item_id": "580",
    "error": "The requested qty is not available"
}
```

### 3.2 decrement quantity request (/:locale/rest/V1a/customer/changeQty/)

- ENDPOINT - /:locale/rest/V1a/customer/changeQty/
- EXAMPLE - /uk/rest/V1a/customer/changeQty/
- TYPE: POST
- REQUEST BODY
- IF CUSTOMER AUTHORIZED REQUEST HEADER MUST INCLUDE Bearer Token, example

```
Authorization:
Bearer eyJraWQiOiIxIiwiYWxnIjoiSFMyNTYifQ.eyJ1aWQiOjEsInV0eXBpZCI6MywiaWF0IjoxNzEwNDkzOTg5LCJleHAiOjE3MTA0OTc1ODl9.eubIKzcqHVb4B18_wc1Z7JoehQGw4xQ7isLUPRbJnM4
```

- REQUEST BODY
```
{
    "quoteNum": 196, | Number
    "itemId": 580, | Number
    "qty": "-1" | String
}
```

#### response type for success decrementing

```
{
    "status": "fulfilled",
    "totalPrice": 478,
    "authorized": true,
    "items_count": 3,
    "products_count": 4,
    "customer_id": 1,
    "quote_id": 196,
    "items": [
        {
            "product_id": 1,
            "product_name": "Пов'язка BluKids Retro Baby",
            "product_sku": "4779679",
            "product_url": "https://m2amida.antoshka.ua/uk/pov-jazka-blukids-retro-baby.html",
            "product_price": 101,
            "qty": 1,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/7/4779679_01.jpg",
            "item_id": 578,
            "in_stock": true
        },
        {
            "product_id": 3,
            "product_name": "Набір аксесуарів Coralico Ніжна квітка, 3 шт",
            "product_sku": "4696127",
            "product_url": "https://m2amida.antoshka.ua/uk/nabir-aksesuariv-coralico-nizhna-kvitka-3-sht.html",
            "product_price": 139,
            "qty": 1,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/6/4696127.jpg",
            "item_id": 579,
            "in_stock": true
        },
        {
            "product_id": 2,
            "product_name": "Набір аксесуарів для волосся Coralico Amour 4 шт",
            "product_sku": "4683549",
            "product_url": "https://m2amida.antoshka.ua/uk/nabir-aksesuariv-dlja-volossja-coralico-amour-4-sht.html",
            "product_price": 119,
            "qty": 2,
            "image_url": "https://m2amida.antoshka.ua/media/catalog/product/cache/eebe3437270609ebda2061ae8ddd8ce0/4/6/4683549_04.jpg",
            "item_id": 580,
            "in_stock": true
        }
    ]
}
```

### 3.3 delete item from cart (/:locale/rest/V1a/cart/delete/item/)
- ENDPOINT :  /:locale/rest/V1a/cart/delete/item/
- EXAMPLE : /uk/rest/V1a/cart/delete/item/
- TYPE: POST
- REQUEST EXAMPLE :
```
{
    "quoteNum": "196", | String
    "itemId": "578" | String
}
```

- RESPONSE WITH SUCCESS DELITING
```
{
    "status": "fulfilled",
    "totalPrice": 139,
    "authorized": true,
    "items_count": 1,
    "products_count": 1,
    "customer_id": 1,
    "quote_id": 196,
    "items": [
        {
            "product_id": 3,
            "product_name": "\u041d\u0430\u0431\u0456\u0440 \u0430\u043a\u0441\u0435\u0441\u0443\u0430\u0440\u0456\u0432 Coralico \u041d\u0456\u0436\u043d\u0430 \u043a\u0432\u0456\u0442\u043a\u0430, 3 \u0448\u0442",
            "product_sku": "4696127",
            "product_url": "https:\/\/m2amida.antoshka.ua\/uk\/nabir-aksesuariv-coralico-nizhna-kvitka-3-sht.html",
            "product_price": 139,
            "qty": 1,
            "image_url": "https:\/\/m2amida.antoshka.ua\/media\/catalog\/product\/cache\/eebe3437270609ebda2061ae8ddd8ce0\/4\/6\/4696127.jpg",
            "item_id": 579,
            "in_stock": true
        }
    ],
    "success": true
}
```