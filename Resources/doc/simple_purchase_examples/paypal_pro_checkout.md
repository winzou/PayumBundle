# Paypal pro checkout

Steps:

* [Download libraries](#download-libraries)
* [Configure context](#configure-context)
* [Prepare payment](#prepare-payment)

_**Note**: We assume you followed all steps in [get it started](../get_it_started.md) and your basic configuration same as described there._

## Download libraries

Run the following command:

```bash
$ php composer.phar require "payum/paypal-pro-checkout-nvp:*@stable"
```

## Configure context

```yaml
#app/config/config.yml

payum:
    contexts:
        your_context_here:
            paypal_pro_checkout_nvp:
                api:
                    options:
                        username: 'EDIT ME'
                        password: 'EDIT ME'
                        partner:  'EDIT ME'
                        vendor:   'EDIT ME'
                        sandbox: true
            storages:
                Acme\PaymentBundle\Entity\PaymentDetails:
                    doctrine:
                        driver: orm
```

_**Attention**: You have to changed `your_payment_name` to something more descriptive and domain related, for example `post_a_job_with_paypal`._

## Prepare payment

Now we are ready to prepare the payment. Here we set price, currency, cart items details and so.
Please note that you have to set details in the payment gateway specific format.

```php
<?php
//src/Acme/PaymentBundle/Controller
namespace AcmeDemoBundle\Controller;

use Payum\Core\Security\SensitiveValue;
use Symfony\Component\HttpFoundation\Request;

class PaymentController extends Controller
{
    public function preparePaypalProCheckoutPaymentAction(Request $request)
    {
        $paymentName = 'your_payment_name';

        $storage = $this->get('payum')->getStorageForClass(
            'Acme\PaymentBundle\Entity\PaymentDetails',
            $paymentName
        );

        /** @var \Acme\PaymentBundle\Entity\PaymentDetails $paymentDetails */
        $paymentDetails = $storage->createModel();
        $paymentDetails['acct'] = new SensitiveValue('5105105105105100');
        $paymentDetails['cvv2'] = new SensitiveValue('123');
        $paymentDetails['expDate'] = new SensitiveValue('1214');
        $paymentDetails['amt'] = 1;
        $paymentDetails['currency'] = 'USD';
        $storage->updateModel($paymentDetails);

        $captureToken = $this->get('payum.security.token_factory')->createCaptureToken(
            $paymentName,
            $paymentDetails,
            'acme_payment_done' // the route to redirect after capture;
        );

        return $this->forward('PayumBundle:Capture:do', array(
            'payum_token' => $captureToken,
        ));
    }
}
```

_**Note**: The sensitive value object ensures that any valuable info will not be saved accidentally somewhere._

That's it. After the payment done you will be redirect to `acme_payment_done` action.
Check [this chapter](../purchase_done_action.md) to find out how this done action could look like.

## Next Step

* [Purchase done action](../purchase_done_action.md).
* [Configuration reference](../configuration_reference.md).
* [Back to examples list](../simple_purchase_examples.md).
* [Back to index](../index.md).