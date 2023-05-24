# DomainExpirationDate
Use GoDaddy's API to fill the expiration date of domain names listed in a Google Sheet

To use Google Apps Script, PHP, and GoDaddy's API together, follow these steps:

1. Create a PHP script (GDAPI.php) on your web server to fetch the domain expiration date using the GoDaddy API. Replace `YOUR_GODADDY_API_KEY` and `YOUR_GODADDY_API_SECRET` with your actual API credentials:

```php
<?php
header('Content-Type: text/plain');

$godaddyApiKey = 'YOUR_GODADDY_API_KEY';
$godaddyApiSecret = 'YOUR_GODADDY_API_SECRET';

function getDomainExpirationDate($domain, $apiKey, $apiSecret) {
    $url = "https://api.godaddy.com/v1/domains/$domain";
    $headers = [
        "Authorization: sso-key $apiKey:$apiSecret"
    ];

    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    $response = curl_exec($ch);
    curl_close($ch);

    $jsonResponse = json_decode($response, true);
    return $jsonResponse['expires'];
}

$domain = $_GET['domain'];
$expirationDate = getDomainExpirationDate($domain, $godaddyApiKey, $godaddyApiSecret);
echo $expirationDate;
?>
```

2. In your Google Sheet, click on "Extensions" in the menu, then select "Apps Script."

3. In the Apps Script editor, replace the existing code with the following code. Replace `https://yourwebsite.com/GDAPI.php` with the actual URL of your GDAPI.php file:

```javascript
function getExpirationDate(domain) {
  var url = 'https://yourwebsite.com/GDAPI.php?domain=' + encodeURIComponent(domain);
  var response = UrlFetchApp.fetch(url);
  var expirationDate = response.getContentText();
  return expirationDate;
}
```

4. Save the script by clicking the floppy disk icon or pressing `Ctrl + S`. Close the Apps Script editor.

5. In your Google Sheet, enter the domain name in cell A1 and use the following formula in cell B1:

```
=getExpirationDate(A1)
```

This formula will call the custom function to fetch the expiration date from your external PHP page using the GoDaddy API and display the result in cell B1.

Please note that Google Apps Script has certain limitations and quotas, such as a maximum execution time of 6 minutes per execution. You can find more details on these limitations in the [Google Apps Script documentation](https://developers.google.com/apps-script/guides/services/quotas).

## Security

To secure your PHP script and prevent unauthorized access, you can implement one or more of the following methods:

1. Use an API key:
Generate a unique API key and include it in your Google Apps Script function when making a request to your PHP script. Then, in your PHP script, check if the provided API key matches the expected value.

In your PHP script (GDAPI.php), add the following code at the beginning:

```php
$expectedApiKey = 'YOUR_UNIQUE_API_KEY';
$providedApiKey = $_GET['api_key'];

if ($providedApiKey !== $expectedApiKey) {
    http_response_code(403);
    echo 'Forbidden: Invalid API key';
    exit;
}
```

In your Google Apps Script function, update the `url` variable to include the `api_key` parameter:

```javascript
var url = 'https://yourwebsite.com/GDAPI.php?domain=' + encodeURIComponent(domain) + '&api_key=YOUR_UNIQUE_API_KEY';
```

Replace `YOUR_UNIQUE_API_KEY` with the same API key you set in your PHP script.

2. Restrict access by IP address:
If you know the IP addresses from which your script will be called, you can restrict access to those IPs in your PHP script.

In your PHP script (GDAPI.php), add the following code at the beginning:

```php
$allowedIPs = ['IP_ADDRESS_1', 'IP_ADDRESS_2']; // Replace with your allowed IP addresses
$clientIP = $_SERVER['REMOTE_ADDR'];

if (!in_array($clientIP, $allowedIPs)) {
    http_response_code(403);
    echo 'Forbidden: Access restricted';
    exit;
}
```

3. Use .htaccess (for Apache servers):
If you are using an Apache server, you can create or edit the .htaccess file in the directory where your PHP script is located to restrict access to specific IP addresses or require authentication.

To restrict access by IP address, add the following to your .htaccess file:

```
<Files "GDAPI.php">
    Order Deny,Allow
    Deny from all
    Allow from IP_ADDRESS_1
    Allow from IP_ADDRESS_2
</Files>
```

Replace `IP_ADDRESS_1` and `IP_ADDRESS_2` with your allowed IP addresses.

To require authentication, add the following to your .htaccess file:

```
<Files "GDAPI.php">
    AuthType Basic
    AuthName "Restricted Access"
    AuthUserFile /path/to/your/.htpasswd
    Require valid-user
</Files>
```

Replace `/path/to/your/.htpasswd` with the path to your .htpasswd file containing the username and password for authentication. You can create an .htpasswd file using an online generator like [htpasswd generator](https://www.htaccesstools.com/htpasswd-generator/).

These methods can be used individually or in combination to secure your PHP script and prevent unauthorized access.
