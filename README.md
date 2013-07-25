laravel-paypalpayment
=====================

laravel-paypalpayment is simple package  help you process   direct credit card payments, stored credit card payments and PayPal account payments with your L4 projects using paypal REST API SDK.
Installation
=============
Install this package through Composer. To your `composer.json` file, add:

```js
"require-dev": {
	"anouar/paypalpayment": "dev-master"
}
```

Next, run `composer update` to download it.

Add the service provider to `app/config/app.php`, within the `providers` array.

```php
'providers' => array(
	// ...

	'Anouar\Paypalpayment\PaypalpaymentServiceProvider',
)
```

Finally, add the alias to `app/config/app.php`, within the `aliases` array.

```php
'aliases' => array(
	// ...

	'Paypalpayment'	  => 'Anouar\Paypalpayment\Facades\PaypalPayment',
)
```
Now go to `vendor\anouar\paypalpayment\src\Anouar\Paypalpayment\sdk_config.ini`

Set your SDK configuration `acct1.ClientId` and `acct1.ClientSecret` , set the `service.EndPoint` to the mode that you want , by default it set to testing mode which is`service.EndPoint="https://api.sandbox.paypal.com"`.If you were going  live , make sure to comment the sandbox mode and uncomment the live mode
.If you do not want to use an ini file or want to pick your configuration dynamically, you can use the `$apiContext->setConfig()` method to pass in the configuration.
That's it !!!!!
==============

Example Code
============
Go to your home controller and past the following code  . make sure to confugure the routes like so `Route::get('/','HomeController@index');`
```php
class HomeController extends BaseController {

    /*
    |--------------------------------------------------------------------------
    | Default Home Controller
    |--------------------------------------------------------------------------
    |
    | You may wish to use controllers instead of, or in addition to, Closure
    | based routes. That's great! Here is an example controller method to
    | get you started. To route to this controller, just add the route:
    |
    |   Route::get('/', 'HomeController@showWelcome');
    |
    */
    //protected $layout="master";

    public function index(){

        echo "<pre>";
        /*Use the OAuth request to retrieve an access token for use with your payments calls.
         OAuthTokenCredential($ClientId,$ClientSecret)
         Commerce identity solution that enables your customers to sign in to your web site quickly and securely using their PayPal login credentials.
        */
        /*Sample code:
        $oauthCredential	= Paypalpayment::OAuthTokenCredential('AVJx0RArQzkCCsWC0evZi1SsoO4gxjDkkULQBdmPNBZT4fc14AROUq-etMEY','EH5F0BAxqonVnP8M4a0c6ezUHq-UT-CWfGciPNQOdUlTpWPkNyuS6eDN-tpA');
		$accessToken     	= $oauthCredential->getAccessToken();
        print_r($accessToken);

        exit();*/


        //grap your credentials .Be sure to set your acct1.ClientId and acct1.ClientSecret on sdk_config.ini
        $cred = Paypalpayment::OAuthTokenCredential();
        print_r($cred);


        // ### Address
        // Base Address object used as shipping or billing
        // address in a payment. [Optional]
        $addr= Paypalpayment::Address();
        $addr->setLine1("3909 Witmer Road");
        $addr->setLine2("Niagara Falls");
        $addr->setCity("Niagara Falls");
        $addr->setState("NY");
        $addr->setPostal_code("14305");
        $addr->setCountry_code("US");
        $addr->setPhone("716-298-1822");

        // ### CreditCard
        // A resource representing a credit card that can be
        // used to fund a payment.
        $card = Paypalpayment::CreditCard();
        $card->setType("visa");
        $card->setNumber("4417119669820331");
        $card->setExpire_month("11");
        $card->setExpire_year("2019");
        $card->setCvv2("012");
        $card->setFirst_name("Anouar");
        $card->setLast_name("Abdessalam");
        $card->setBilling_address($addr);

        // ### FundingInstrument
        // A resource representing a Payer's funding instrument.
        // Use a Payer ID (A unique identifier of the payer generated
        // and provided by the facilitator. This is required when
        // creating or using a tokenized funding instrument)
        // and the `CreditCardDetails`
        $fi = Paypalpayment::FundingInstrument();
        $fi->setCredit_card($card);


        // ### Payer
        // A resource representing a Payer that funds a payment
        // Use the List of `FundingInstrument` and the Payment Method
        // as 'credit_card'
        $payer = Paypalpayment::Payer();
        $payer->setPayment_method("credit_card");
        $payer->setFunding_instruments(array($fi));

        // ### Amount
        // Let's you specify a payment amount.
        $amount = Paypalpayment:: Amount();
        $amount->setCurrency("USD");
        $amount->setTotal("10.00");

        // ### Transaction
        // A transaction defines the contract of a
        // payment - what is the payment for and who
        // is fulfilling it. Transaction is created with
        // a `Payee` and `Amount` types
        $transaction = Paypalpayment:: Transaction();
        $transaction->setAmount($amount);
        $transaction->setDescription("This is the payment description.");

        // ### Payment
        // A Payment Resource; create one using
        // the above types and intent as 'sale'
        $payment = Paypalpayment:: Payment();
        $payment->setIntent("sale");
        $payment->setPayer($payer);
        $payment->setTransactions(array($transaction));

        // ### Api Context
        // Pass in a `ApiContext` object to authenticate 
        // the call and to send a unique request id 
        // (that ensures idempotency). The SDK generates
        // a request id if you do not pass one explicitly. 
        $apiContext = Paypalpayment:: ApiContext($cred, 'Request' . time());

        // ### Create Payment
        // Create a payment by posting to the APIService
        // using a valid ApiContext
        // The return object contains the status;
        try {
            $payment->create($apiContext);
        } catch (\PPConnectionException $ex) {
            return "Exception: " . $ex->getMessage() . PHP_EOL;
            var_dump($ex->getData());
            exit(1);
        }

        $response=$payment->toArray();
        echo"<pre>";
        print_r($response);
        //var_dump($payment->getId());

        //print_r($payment->toArray());

    }

}
```
These examples pick the SDK configuration from the sdk_config.ini file.
Conclusion
==========
I hope this package help somone around -_*
