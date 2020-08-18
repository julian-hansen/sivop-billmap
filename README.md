# Process to fix BillMap timing issue.

## Remove the authenticate debit from the yup_script.js

In the following file

shop\curl\yup_script.js

On Line 17 - remove the click handler

    checkAcctBtn.click(function(e) {
        e.preventDefault();
        errorMessage.hide();
        $.ajax({
            url: API + 'debit-request',
            data: getData(),
            type: 'POST',
            dataType: 'JSON'
        })
        .then(function(resp) {
            if (resp.status) {
                tokenInput.show();
                // validateButton.click(validateTransaction); // <=== REMOVE THIS LINE
            } else {
				tokenInput.hide();
                errorMessage.html(resp.error).show();
            }
        })
    })

This function is no longer needed and can be removed

    function validateTransaction(e) {
        e.preventDefault();
        $.ajax({
            url: API + 'authenticate-debit',
            data: {token : tokenInput.val()},
            type: 'POST',
            dataType: 'JSON'
        })
        .then(function(resp) {
            if (resp.status) {
                transactionForm.submit();
            } else {
                alert('Oups ! Une erreur s’est produite.');
            }
        })
    }

The form will now submit normally using the **CONFIRM** button.

In your form processing script (PHP file) you will need to do the following

    ...
    // Create your transaction record here
    ...
    // Include the BillMap API script to complete the transaction with BillMap
    // Define _YUP_API so that the API will run (see api file for details)
    define('_YUP_API', true);
    require_once('/shop/curl/yup/api/billmap-api.php');
    $yup = new YupActionAuthenticateDebit;
    $result = $yup->run();

    // $result will now hold the return from the Authenticate Debit
    // You can use that here to update your database with the status
    // code.


You will need this file added to the YUP API files for the above code to work.Create the file /shop/curl/yup/api/billmap-api.php with the following code

    <?php
    // Prevent direct access. Define _YUP_API in the calling script
    defined('_YUP_API') or die();
    define('YUPAPI_PATH', __DIR__);

    // Include the YUPAPI base class
    require_once(YUPAPI_PATH . '/actions/yup-action.class.php');

    class YupActionAuthenticateDebit extends YupAction {
        public function run() {
            $response = false;

            $data = $this->getData();
            if ($data) {
                $response = $this->authenticate_debit($data);
            }

            return $response;
        }

        private function getData() {
            $data = new stdClass;

            if (isset($_SESSION[YUP_BILLMAP_ID])) {
                $data->BillMapTransactionId = $_SESSION[YUP_BILLMAP_ID];
            } else {
                return false;
            }

            if (isset($_SESSION[YUP_MSISDN])) {
                $data->MSISDN = $_SESSION[YUP_MSISDN];
            } else {
                return false;
            }

            if (isset($_POST['token'])) {
                $data->Token = $_POST['token'];
            } else {
                return false;
            }

            return $data;
        }
    }


The above script will do what the AJAX script was doing before - except this time it will be part of your script so you can control it.


