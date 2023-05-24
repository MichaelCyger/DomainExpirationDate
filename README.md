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
