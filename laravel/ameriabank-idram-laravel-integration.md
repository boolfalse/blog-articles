
## AmeriaBank & IDram V-POS integration to your Laravel (PHP) app

Often, online services do not have convenient documentation for developers. And sometimes difficulties arise when following the existing documentation. In this article, I will share the information I have and my experience about two online services that can help you to have some initial insights and integrate them in your app more easily.

I will describe the steps you need to follow for these two Virtual-POS (point of sale) systems to integrate in your Laravel app:

- [AmeriaBank](https://ameriabank.am/) vPOS integration.
- [IDram](https://www.idram.am/) V-POS integration.

Both of these are frequently used online methods in recent years in the Armenian e-market and not only.

<img src="https://i.imgur.com/Lq4IzL1.png" style="width: 100%;">

### BEFORE THE BEGINNING:

Before starting to integrate any of these pieces of software, contact the official representative of the respective department through one of the contact methods listed on their official website (recommended by phone), who will provide you the credentials and the necessary information you may need.

***

In case of Amerabank representative's agreement, they will create a test environment for your app and you will receive an e-mail of the following form from the employee։

**_AmeriaBank Credentails Email:_**

```text
Below is the real environment data of Ameriabank vPOS payment system integration:

Main address:
https://services.ameriabank.am/VPOS/

Address for receiving information about transactions:
https://payments.ameriabank.am/Admin/webservice/TransactionsInformationService.svc?wsdl

ClientID: ********-****-****-****-************
ClientUsr: **********
ClientPass: ****************

You can see the transactions made on your website with this link:
https://payments.ameriabank.am/admin/clients/

Credentials:
Username: ************
Password: ************

With respect,
vPOS support team
```

**_AmeriaBank Information Email:_**

```text
## vPOS Testing Credentials:

The ConfirmPayment request is required for two-step payments for the implementation of the second stage and withdrawal of money from the buyer's (cardholder's) account.
If you plan to make payments in one round, then there is no need to perform the mentioned function.
Please make the appropriate correction and make test payments.

We inform you that the AmeriaBank VPOS system test environment has been created for you.
A brief description of system integration is attached.
You can also see the mentioned information at the following link:
https://servicestest.ameriabank.am/VPOS/help

In the test environment, the OrderID should be selected from the range 2350301-2350400, and the amount should be 10 AMD.
In the default test system, the ability to make one-step payments, Refund and Cancel functions are provided.
In order to make 2-step payments and related transactions, it is necessary to inform additionally in order to obtain the relevant authority.
It should be noted that the testing is considered carried out, if made through the Rest protocol and at least 5 successful payments with completed status are recorded.
Please confirm with a return email that you have received the data.
And in case of technical questions, you can call or write to us.

## vPOS minimum requirements and technical instructions:

The list of minimum requirements and technical guide for installing vPOS on the website are presented on the website:
https://ecommerce.ameriabank.am/.

Please also provide a demo version of the site that can be viewed.
```

**_AmeriaBank Documents Email:_**

- [vPOS minimum requirements](https://mega.nz/file/bxIVnSxA#btQLXFP-iEKz5GDrSzqYSW7bJS0Y7tHwGo7kevl95aU)
- AmeriaBank vPOS 3.0: [API Protocol Description (Eng)](https://mega.nz/file/v0ZiSR7Z#fx0A5kIfvN3Gff-VFvpoBLah-4ptVIw8B77XxKYFNHg)
- AmeriaBank vPOS 3.0: [API նկարագրություն (Arm)](https://mega.nz/file/r9YQULAK#TWz0JjnU6Jz-jAq7hhDsNrEi-qXG_Q2mDR7aLf_LQcM)
- Ameriabank CJSC POS Service: [Terms and Conditions](https://ameriabank.am/Portals/0/files/Business/pos/POS_Terms_and_Conditions_eng.pdf)

***

Let's firstly integrate **AmeriaBank vPOS 3.0**.

[AmeriaBank vPOS](https://ameriabank.am/en/business/micro/more/other-services/ecommerce) (virtual POS terminal) is a free software tool, which enables the merchant to avoid issues with time-consuming development, configuration, certification and further modification of software. It gives opportunity to have a fast and secure solution for accepting online card payments.

At first, add some environment variables to your **_.env_** file:

```text
AMERIABANK_CLIENT_ID="********-****-****-****-************"
AMERIABANK_USERNAME="**********"
AMERIABANK_PASSWORD="******"
AMERIABANK_SUBDOMAIN="testpayments"

AMERIABANK_TEST_CARD_NUMBER="****************"
AMERIABANK_TEST_CARD_CARDHOLDER="TEST CARD VPOS"
AMERIABANK_TEST_CARD_EXP_DATE="06/26"
AMERIABANK_TEST_CARD_CVV="***"
```

_AMERIABANK_SUBDOMAIN_ may be one of these:

- _testpayments_ - for testing
- _services_ - for production (recommended for live)
- _payments_ - another subdomain

Create a custom **_config/payment.php_** payment-specific configurations file:

```shell
'ameria' => [
    'client_id' => env('AMERIABANK_CLIENT_ID'),
    'username' => env('AMERIABANK_USERNAME'),
    'password' => env('AMERIABANK_PASSWORD'),
    'subdomain' => env('AMERIABANK_SUBDOMAIN', 'testpayments'),
],
```

Add a webhook route to your **routes/web.php**:

```shell
Route::post('/payment/ameria/{code}', 'PaymentController@ameria');
```

**Important**: Don't forget to disable any firewalls or middlewares that could block the incoming webhook-requests to your app.

Refresh caches after modifying configuration files and routes:

```shell
php artisan optimize
```

Below is the **_app/Http/Controllers/PaymentController_** snippet which can be used to accept payment webhook:

```shell
public function ameria(Request $request, string $code)
{
    $sale_data = [
        'payment_method' => 'card',
        'payment_provider' => 'ameria',
        'payment_status' => 'pending',
        'payment_json' => [
            'request' => null,
            'response' => null,
        ],
    ];
    $error = false;

    if ($request->has('orderID'))
    {
        /* This block is just an example of getting the SALE record
        you need to be already have created in your app after successful checkout */
        $sale = Sale::where('payment_method', 'card')
            ->where('code', $request['orderID'])
            ->first();
        if (empty($sale)) {
            return redirect()->route('errors.404');
        }

        $sale_data['payment_json']['request'] = $request->all();
        if ($request->has('paymentid'))
        {
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, "https://" . config('payment.ameria.subdomain') . ".ameriabank.am/VPOS/api/VPOS/GetPaymentDetails");
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
            curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode([
                "PaymentID" => $request->get('paymentid'),
                "Username" => config('payment.ameria.username'),
                "Password" => config('payment.ameria.password'),
            ], JSON_UNESCAPED_UNICODE));
            curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
            $response = curl_exec($ch);
            $curlError = curl_error($ch);
            curl_close($ch);

            $sale_data['payment_json']['response'] = $response;

            if ($curlError) {
                $error = true;
            } else {
                switch ($request->get('respcode')) {
                    case "00":
                        break;
                    default:
                        $error = true;
                }
            }
        } else {
            $error = true;
        }

        $sale_data['payment_status'] = $error ? 'failed' : 'paid';

        // This block is just an example of updating the SALE record 
        $sale->update($sale_data);

        if ($error) {
            // You can return and show some error message
            return redirect()->route('front.main');
        } else {
            // This is just an example of creating some PDF for report
            $pdf = $this->paymentService->generateSaleInvoice($sale);

            // This is just an example of emailing the report
            $this->paymentService->sendSaleEmail($sale, $pdf);

            return redirect()->route('front.order_success', ['code' => $sale->code]);
        }
    } else {
        return redirect()->route('errors.404');
    }
}
```

You can read more about the system logic on [**_VPOS Web API Help Page_**](https://servicestest.ameriabank.am/VPOS/help).

In the end, just in case I will put here some links that may be useful:

- [PaymentService Service](https://payments.ameriabank.am/webservice/PaymentService.svc)
- [AmeriaBank VPOS Laravel Package](https://github.com/ayvazyan10/AmeriaBankvpos)
- [Response codes of Ameria Bank REST API. Maybe also can be used for ARCA response codes](https://gist.github.com/hos/4ed551472aeb6be0ab4b29170d3fd726)
- [Ameria Bank vPos API](https://github.com/vahanspetrosyan/omnipay-vpos-ameriabank)
- [The VPOS SDK for Ameria Bank](https://github.com/hos/ameria-sdk-js)

***

Now let's integrate **IDram**.

[**_IDram V-POS (virtual POS terminal)_**](https://idbank.am/en/business/instruments/trade-finance/v-pos-virtual-pos-terminal-0/) enables trading points to accept card payments online quickly and securely. V-POS terminals enable the customer to expand product or service sales by accepting payments in the Internet environment 24/7 from anywhere in the world.

At first, add some environment variables to your **_.env_** file:

```shell
IDRAM_SECRET_KEY="**************************************"
IDRAM_EDP_REC_ACCOUNT="*********"
```

Add necessary webhook routes to your **_routes/web.php_**:

```shell
Route::match(['GET', 'POST', 'PUT'], '/payment/idram', 'PaymentController@idram');
```

**Important**: Don't forget to disable any firewalls or middlewares that could block the incoming webhook-requests to your app.

In the **_config/payment.php_** add these payment-specific configurations:

```shell
'idram' => [
    'secret_key' => env('IDRAM_SECRET_KEY'),
    'edp_rec_account' => env('IDRAM_EDP_REC_ACCOUNT'),
],
```

Refresh caches after modifying configuration files and routes:

```shell
php artisan optimize
```

Below is the **_app/Http/Controllers/PaymentController_** snippet which can be used to accept payment webhooks:

```shell
public function idram(Request $request)
{
    /*
    $request_body = [];
    // for both requests
    $request_body['EDP_BILL_NO'] = isset($_REQUEST['EDP_BILL_NO']) ? $_REQUEST['EDP_BILL_NO'] : null;
    $request_body['EDP_REC_ACCOUNT'] = isset($_REQUEST['EDP_REC_ACCOUNT']) ? $_REQUEST['EDP_REC_ACCOUNT'] : null;
    $request_body['EDP_AMOUNT'] = isset($_REQUEST['EDP_AMOUNT']) ? $_REQUEST['EDP_AMOUNT'] : null;
    // 1. Order Authenticity
    $request_body['EDP_PRECHECK'] = isset($_REQUEST['EDP_PRECHECK']) ? $_REQUEST['EDP_PRECHECK'] : null;
    // 2. Payment Confirmation
    $request_body['EDP_PAYER_ACCOUNT'] = isset($_REQUEST['EDP_PAYER_ACCOUNT']) ? $_REQUEST['EDP_PAYER_ACCOUNT'] : null;
    $request_body['EDP_TRANS_ID'] = isset($_REQUEST['EDP_TRANS_ID']) ? $_REQUEST['EDP_TRANS_ID'] : null;
    $request_body['EDP_TRANS_DATE'] = isset($_REQUEST['EDP_TRANS_DATE']) ? $_REQUEST['EDP_TRANS_DATE'] : null;
    $request_body['EDP_CHECKSUM'] = isset($_REQUEST['EDP_CHECKSUM']) ? $_REQUEST['EDP_CHECKSUM'] : null;
    */
    
    // Idram Payment System provide it
    $secret = config('payment.idram.secret_key');
    // Idram Payment System provide it
    $edp_rec_account = config('payment.idram.edp_rec_account');

    // 1. Order Authenticity
    if(isset($_REQUEST['EDP_PRECHECK']) &&
        isset($_REQUEST['EDP_BILL_NO']) &&
        isset($_REQUEST['EDP_REC_ACCOUNT']) &&
        isset($_REQUEST['EDP_AMOUNT']))
    {
        if($_REQUEST['EDP_PRECHECK'] == "YES") {
            if($_REQUEST['EDP_REC_ACCOUNT'] == $edp_rec_account) {
                $bill_no = $_REQUEST['EDP_BILL_NO'];
                // this code checks if $bill_no exists in your system orders if exists then echo OK otherwise nothing

                /* This block is just an example of getting the SALE record
                you need to be already have created in your app after successful checkout */
                $sale = Sale::where('code', $bill_no)
                    // ->where('payment_provider', 'idram')
                    ->whereIn('payment_status', ['defined', 'failed', 'pending']) // pending - when already someone want to pay with idram
                    ->first();

                if ($sale) {
                    $sale->update([
                        'payment_method' => 'idram',
                        'payment_provider' => 'idram',
                        'payment_status' => 'pending',
                        'payment_json' => $request->all(),
                    ]);

                    echo "OK"; die;
                }
            }
        }
    }

    // 2. Payment Confirmation
    if(isset($_REQUEST['EDP_PAYER_ACCOUNT']) &&
        isset($_REQUEST['EDP_BILL_NO']) &&
        isset($_REQUEST['EDP_REC_ACCOUNT']) &&
        isset($_REQUEST['EDP_AMOUNT']) &&
        isset($_REQUEST['EDP_TRANS_ID']) &&
        isset($_REQUEST['EDP_CHECKSUM']))
    {
        $txtToHash =
            $edp_rec_account . ":" .
            $_REQUEST['EDP_AMOUNT'] . ":" .
            $secret . ":" .
            $_REQUEST['EDP_BILL_NO'] . ":" .
            $_REQUEST['EDP_PAYER_ACCOUNT'] . ":" .
            $_REQUEST['EDP_TRANS_ID'] . ":" .
            $_REQUEST['EDP_TRANS_DATE'];

        if(strtoupper($_REQUEST['EDP_CHECKSUM']) != strtoupper(md5($txtToHash)))
        {
            $bill_no = $_REQUEST['EDP_BILL_NO'];
            
            // Write your code here to handle the payment fail

            /* This block is just an example of getting the SALE record
            you need to be already have created in your app after successful checkout */
            Sale::where([
                ['code', '=', $bill_no],
                ['payment_status', '=', 'pending'],
                // ['payment_provider', '=', 'idram'],
            ])->update([
                'payment_method' => 'idram',
                'payment_provider' => 'idram',
                'payment_status' => 'failed',
                'payment_json' => $request->all(),
            ]);

            // echo("FAIL!");
        }
        else
        {
            $bill_no = $_REQUEST['EDP_BILL_NO'];
            // please, write your code here to handle the payment success

            /* This block is just an example of getting the SALE record
            you need to be already have created in your app after successful checkout */
            $sale = Sale::where([
                ['code', '=', $bill_no],
                ['payment_status', '=', 'pending'],
                // 'payment_provider' => 'idram',
            ])->first();

            if ($sale) {
                // This block is just an example of updating the SALE record
                $sale->update([
                    'payment_method' => 'idram',
                    'payment_provider' => 'idram',
                    'payment_status' => 'paid',
                    'payment_json' => $request->all(),
                ]);

                // This is just an example of creating some PDF for report
                $pdf = $this->paymentService->generateSaleInvoice($sale);
                
                // This is just an example of emailing the report
                $this->paymentService->sendSaleEmail($sale, $pdf);

                echo "OK"; die;
            }
        }
    }

    if (isset($_REQUEST['EDP_BILL_NO'])) {
        $sale = Sale::where([
            ['code', '=', $_REQUEST['EDP_BILL_NO']],
            ['payment_status', '=', 'paid'],
        ])->first();

        if ($sale) {
            return redirect()->route('front.order_success', ['code' => $sale->code]);
        } else {
            // Redirect to error page with reason message
            return redirect()->route('front.main');
        }
    }
}
```

As you may have already noticed, in the **_idram_** method we have implemented 2 separate logics for 2 cases: _Order Authenticity_ and _Payment Confirmation_, so that we can accept that two main incoming requests during the payment process. You can read more about that system logic on Idram Payment System [**documentation**](https://www.slideshare.net/iDramAPI/idram-merchant-api-idram-payment-system-merchant-interface-description) official document.
Alternatively, check for this [**link**](https://mega.nz/file/79411ZKT#CvlbhEgMRCMiQgBN0Rjij_RasB1RRoijo1lKlqes_QM).

In the end, just in case I will put here some links that may be useful:

- [IdramMerchantPayment](https://github.com/karapetyangevorg/IdramMerchantPayment) (the old one [here](https://github.com/gagikmartirosyan/IdramMerchantPayment))
- [Omnipay iDram](https://github.com/ptuchik/omnipay-idram)

**That's it.**

***

If you liked this article, feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [**GitHub**](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [**boolfalse.com**](https://boolfalse.com/)

Thank you !!!
