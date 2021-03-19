# SMS-Spoofing
Security research into SMS Spoofing
  
### Example 1    
Here we look at using SMPP Servers to Spoof a From Name of our SMS, this example uses the Free ClickSend API.
  
They only allow 1 account so be careful - i was blocked after testing.  
``` 
 <?php

echo sendSMS("+614","Test Message", true);

function sendSMS($to, $msg, $testing = false)
{

// Get Account from List
$acc = getAccount();
$username = $acc[0];
$key = $acc[1];

if($testing)
{
$to = "+61411111111";
}

// Send SMS
$result = clickSend($username, $key, $to, $msg, "TEST");

// Update Quota in Account List
if (!$testing)
{
$acc = getAccount(true);
}
return $result;
}


// ClickSend SMS Function
function clickSend($username, $key, $to, $msg, $senderID = "Anon")
{

$url = urlDecode("https://api.clicksend.com/http/v2/send.php?method=http&username=$username&key=$key");
  $data = '&to=' . $to .'&message=' . $msg . '&senderid=' . $senderID;
  $ch = curl_init(); 
  curl_setopt($ch, CURLOPT_URL, $url);
  curl_setopt($ch, CURLOPT_POST, 1); 
  curl_setopt($ch, CURLOPT_POSTFIELDS, $data); 
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, 0); 
  $result = curl_exec($ch);
  $error = curl_error($ch);
  curl_close($ch);
 if (strlen($error == 0))
  {
    return $result;
  }
  else
 {
   return $error;
 }
}



// CSV Account List Manager
function getAccount($consumeQuota = false)
{
$filename = 'accs.list';
$csvarray = []; 
$selected = array();

if (($h = fopen("{$filename}", "r")) !== FALSE) 
{
  while (($data = fgetcsv($h, 1000, ",")) !== FALSE) 
  {
    $csvarray[] = $data;		
  }
  fclose($h);
}

$fp = fopen($filename, 'w');

foreach ($csvarray as $fields) {

    $username = $fields[0];
    $key = $fields[1];
    $quota = $fields[2];
      if ($quota >= 1)
          {
            if($consumeQuota)
              {
                 $fields[2] = $quota -1;
              }
            $selected = $fields;
          }

    fputcsv($fp, $fields);
    $lines++;
}
fclose($fp);
return $selected;
}

?>
```   
 
### Example 2   
This example requires RAW PDU SMS Delivery, you can find RAW SMS PDU Modification Support on old HTC Android Devices and some old Huawei devices.  
    
```   
The SMS-DELIVER TPDU consists of 7 fields and the message’s content :

SCA (Service-Center-Address)
PDU Type
TP-OA (TP-Originator-Address)
TP-PID (TP-Protocol-Identifier)
TP-DCS (TP-Data-Coding-Scheme)
TP-SCTS (TP-Service-Center-Time-Stamp)
TP-UDL (TP-User-Data-Length)
TP-UD (TP-User-Data)   
 
Please refer to Structure Documentation of PDU Payloads.   
https://bluesecblog.wordpress.com/2016/11/15/sms-deliver-tpdu-structure/   
 
```
