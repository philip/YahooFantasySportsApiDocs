
# Getting Started

## OAuth

If you're going to use the Fantasy Sports APIs, you're going to have to get a bit familiar with OAuth. OAuth is the authentication mechanism for these services that allows users to grant you permission to make requests on their behalf. Many other Yahoo! services use OAuth, and thus all of the underlying details are explained in exhaustive detail in our primary OAuth documentation. Of particular interest is the OAuth Authorization Flow, which explains where each request is made and where the user needs to get involved.

However, constructing OAuth flows from scratch is complicated and easy to get wrong. It's often easier to use existing libraries, which are available for most languages on the OAuth.net Code page.

## Registering Your Application

To work with OAuth and Yahoo! services, you also must register your application with the Yahoo! Developer Network. When you register your application, you define a scope of Yahoo! services that your application will need access to, as well as the basic descriptive information that will be presented to users of your application when they're asked to grant you permissions. You will be given a consumer key and secret value that will need to be fed into OAuth requests that you generate. You should be sure to keep these values secret, as anyone with access to them could masquerade as your application.

To create a new OAuth application to use with the Fantasy Sports APIs, you should go through the New API Key flow on YDN. Be sure to specify that you need access to private user data, and select either Read or Read/Write access for Fantasy Sports.

## PHP Sample Code

### Basic OAuth Library Use

While everyone will have their favorite language to use when writing applications using the Fantasy Sports APIs, hopefully this PHP example will still serve as a useful reference. It tries to do several interesting things: keeping track of (and potentially refreshing) access tokens for future reuse, facilitating the interactive user authentication flow, and actually making a request based on any tokens retrieved. In order to use this code, you must have the default PHP OAuth extension installed, and you will need to fill in your own consumer key and secret where specified. You should then be able to execute the PHP script from the command line.

```
<?php
// **** POTENTIAL CONFIGURATION STARTS HERE ****

// NOTE: If you don't have the OAuth extension hooked into PHP, you may need
//  to include it here.

// MODIFY: Insert whichever URL you'd like to try below. By default, the 
//  following URL will try to pull out the NFL teams for the logged-in user
$url = 'http://fantasysports.yahooapis.com/fantasy/v2/users;use_login=1/games;game_keys=nfl/teams'; 
$scope = 'test';

// MODIFY: Insert your own consumer key and secret here!
$consumer_data = array();
$consumer_data['test']['key'] = '<INSERT CONSUMER KEY HERE>';
$consumer_data['test']['secret'] = '<INSERT CONSUMER SECRET HERE>';


// **** MAIN PROGRAM START HERE ****

$consumer_key = $consumer_data[$scope]['key'];
$consumer_secret = $consumer_data[$scope]['secret'];

// By default, try to store token information in /tmp folder
$token_file_name = '/tmp/oauth_data_token_storage_' . $consumer_key . '.out';

$access_token = NULL;
$access_secret = NULL;
$access_session = NULL;
$access_verifier = NULL;
$store_access_token_data = false;

if( file_exists( $token_file_name ) && 
    $tok_fh = fopen( $token_file_name, 'r' ) ) {

  $invalid_file = false;

  // Get first line: access token
  $access_token = fgets( $tok_fh );
  if( $access_token ) {
    // Get next line: access secret
    $access_secret = fgets( $tok_fh );
    if( $access_secret ) {
      // Get next line: access session handle
      $access_session = fgets( $tok_fh );
      if( ! $access_session ) {
        $invalid_file = true;
      }
    } else {
      $invalid_file = true;
    }
  } else {
    $invalid_file = true;
  }

  if( $invalid_file ) {
    print "File did not seem to be formatted correctly -- needs 3 lines with access token, secret, and session handle.\n";
    $access_token = NULL;
    $access_secret = NULL;
    $access_session = NULL;
  } else {
    print "Got access token information!\n";

    $access_token = rtrim( $access_token );
    $access_secret = rtrim( $access_secret );
    $access_session = rtrim( $access_session );

    print " Token: ${access_token}\n";
    print " Secret: ${access_secret}\n";
    print " Session Handle: ${access_session}\n\n";
  }
  
  // Done with file, close it up
  fclose( $tok_fh );

} else {
  print "Couldn't open ${token_file_name}, assuming we need to get a new request token.\n";
}

// 1. See if we have a stored access token/secret/session. If so, try to use
//    that token.
if( $access_token ) {

  $o = new OAuth( $consumer_key, $consumer_secret,
                  OAUTH_SIG_METHOD_HMACSHA1, OAUTH_AUTH_TYPE_URI );
  $o->enableDebug();

  $auth_failure = false;
  
  // Try to make request using stored token
  try {
    $o->setToken( $access_token, $access_secret );
    if( $o->fetch( $url ) ) {
      print "Got data from API:\n\n";
      print $o->getLastResponse() . "\n\n";

      print "Successful!\n";
      exit;
    } else {
      print "Couldn'\t fetch\n";
    }
  } catch( OAuthException $e ) {
    print 'Error: ' . $e->getMessage() . "\n";
    print 'Error Code: ' . $e->getCode() . "\n";
    print 'Response: ' . $e->lastResponse . "\n";

    if( $e->getCode() == 401 ) {
      $auth_failure = true;
    }
  }


  // 2. If we get an auth error, try to refresh the token using the session.
  if( $auth_failure ) {
    
    try {
      $response = $o->getAccessToken( 'https://api.login.yahoo.com/oauth/v2/get_token', $access_session, $access_verifier );
    } catch( OAuthException $e ) {
      print 'Error: ' . $e->getMessage() . "\n";
      print 'Response: ' . $e->lastResponse . "\n";

      $response = NULL;
    }

    print_r( $response );

    if( $response ) {
      $access_token = $response['oauth_token'];
      $access_secret = $response['oauth_token_secret'];
      $access_session = $response['oauth_session_handle'];
      $store_access_token_data = true;

      print "Was able to refresh access token:\n";
      print " Token: ${access_token}\n";
      print " Secret: ${access_secret}\n";
      print " Session Handle: ${access_session}\n\n";

    } else {
      
      $access_token = NULL;
      $access_secret = NULL;
      $access_session = NULL;
      print "Unable to refresh access token, will need to request a new one.\n";
    }
  }
}

// 3. If none of that worked, send the user to get a new token
if( ! $access_token ) {
  
  print "Better try to get a new access token.\n";
  $o = new OAuth( $consumer_key, $consumer_secret,
                  OAUTH_SIG_METHOD_HMACSHA1, OAUTH_AUTH_TYPE_URI );
  $o->enableDebug();

  $request_token = NULL;

  try {
    $response = $o->getRequestToken( "https://api.login.yahoo.com/oauth/v2/get_request_token", 'oob' );
    $request_token = $response['oauth_token'];
    $request_secret = $response['oauth_token_secret'];

    print "Hey! Go to this URL and tell us the verifier you get at the end.\n";
    print ' ' . $response['xoauth_request_auth_url'] . "\n";

  } catch( OAuthException $e ) {
    print $e->getMessage() . "\n";
  }

  // Wait for input, then try to use it to get a new access token.
  if( $request_token && $request_secret ) {
    print "Type the verifier and hit enter...\n";
    $verifier = fgets( STDIN );
    $verifier = rtrim( $verifier );
    
    print "Here's the verifier you gave us: ${verifier}\n";
    
    try {
      $o->setToken( $request_token, $request_secret );
      $response = $o->getAccessToken( 'https://api.login.yahoo.com/oauth/v2/get_token', NULL, $verifier );

      print "Got it!\n";
      $access_token = $response['oauth_token'];
      $access_secret = $response['oauth_token_secret'];
      $access_session = $response['oauth_session_handle'];
      $store_access_token_data = true;
      print " Token: ${access_token}\n";
      print " Secret: ${access_secret}\n";
      print " Session Handle: ${access_session}\n\n";


    } catch( OAuthException $e ) {
      print 'Error: ' . $e->getMessage() . "\n";
      print 'Response: ' . $e->lastResponse . "\n";
      print "Shoot, couldn't get the access token. :(\n";
    }
  }

}

if( $access_token ) {

  // Try to make request using stored token
  try {
    $o->setToken( $access_token, $access_secret );
    if( $o->fetch( $url ) ) {
      print "Got data from API:\n\n";
      print $o->getLastResponse() . "\n\n";

      print "Successful!\n";
    } else {
      print "Couldn'\t fetch\n";
    }
  } catch( OAuthException $e ) {
    print 'Error: ' . $e->getMessage() . "\n";
    print 'Error Code: ' . $e->getCode() . "\n";
    print 'Response: ' . $e->lastResponse . "\n";
  }
}

// 4. Rewrite token information if necessary
if( $store_access_token_data ) {

  print "Looks like we need to store access token data! Doing that now.\n";

  $tok_fh = fopen( $token_file_name, 'w' );
  if( $tok_fh ) {
    fwrite( $tok_fh, "${access_token}\n" );
    fwrite( $tok_fh, "${access_secret}\n" );
    fwrite( $tok_fh, "${access_session}\n" );
    
    fclose( $tok_fh );
  } else {
    print "Hm, couldn't open file to write back access token information.\n";
  }
}
?>
```
        
### Full OAuth Flow without Libraries

If the PHP library described above doesn't fit your needs, you may feel like implementing the flow yourself. It's mildly tricky and you may run into common issues like not sorting the parameters correctly, or not encoding the various parts of the URL at the right time. The following script is an example of doing absolutely everything from scratch, and may be a useful guide for you.

```
<?php
// **** POTENTIAL CONFIGURATION STARTS HERE ****

// MODIFY: Insert your own consumer key and secret here!
$consumer_data = array();
$consumer_data['test']['key']    = '<INSERT CONSUMER KEY HERE>';
$consumer_data['test']['secret'] = '<INSERT CONSUMER SECRET HERE>';


// **** HELPER FUNCTIONS START HERE ****

///////////////////////////////////////////////////////////////////////////////
//  FUNCTION _make_signed_request
/// @brief Helper function to make a signed OAuth request. Only allows GET 
///        requests at the moment. Will add on standard OAuth params, but
///        you may need to fill in non-generic ones ahead of time.
///
/// @param[in]  $consumer_key      Application consumer key
/// @param[in]  $consumer_secret   Application consumer secret
/// @param[in]  $token             Token (request or access token)
/// @param[in]  $token_secret      Token secret
/// @param[in]  $signature_method  'PLAINTEXT' or 'HMAC-SHA1'
/// @param[in]  $url               URL to make request to
/// @param[in]  $params            Array of key=>val for params. Don't
///                                urlencode ahead of time, we'll do that here.
///////////////////////////////////////////////////////////////////////////////
function _make_signed_request( $consumer_key, $consumer_secret, $token, $token_secret, $signature_method, $url, $params = array() ) {

  // Only support GET in this function
  $method = 'GET';

  $signature_method = strtoupper( $signature_method );
  if( $signature_method != 'PLAINTEXT' && $signature_method != 'HMAC-SHA1' ) {
    print "Invalid signature method: ${signature_method}\n";
    return false;
  }

  $oauth_nonce = rand( 0, 999999 );
  $oauth_timestamp = time();
  $oauth_version = '1.0';

  $params['oauth_consumer_key'] = $consumer_key;
  $params['oauth_nonce'] = $oauth_nonce;
  $params['oauth_signature_method'] = $signature_method;
  $params['oauth_timestamp'] = $oauth_timestamp;
  $params['oauth_version'] = $oauth_version;

  if( $token ) {
    $params['oauth_token'] = $token;
  }
  if( ! $token_secret ) {
    $token_secret = '';
  }
  
  // Params need to be sorted by key
  ksort( $params, SORT_STRING );

  // Urlencode params and generate param string
  $param_list = array();
  foreach( $params as $key => $value ) {
    $param_list[] = urlencode( $key ) . '=' . urlencode( $value );
  }
  $param_string = join( '&', $param_list );
  
  // Generate base string (needed for SHA1)
  $base_string = urlencode( $method ) . '&' . urlencode( $url ) . '&' . 
    urlencode( $param_string );

  // Generate secret
  $secret = urlencode( $consumer_secret ) . '&' . urlencode( $token_secret );
  if( $signature_method == 'PLAINTEXT' ) {
    $signature = $secret;
  } else if( $signature_method == 'HMAC-SHA1' ) {
    $signature = base64_encode( hash_hmac( 'sha1', $base_string, $secret, true ) );
  }
  
  // Append signature
  $param_string .= '&oauth_signature=' . urlencode( $signature );
  $final_url = $url . '?' . $param_string;

  // Make curl call
  $ch = curl_init();
  curl_setopt( $ch, CURLOPT_URL, $final_url );
  curl_setopt( $ch, CURLOPT_AUTOREFERER, 1 );
  curl_setopt( $ch, CURLOPT_RETURNTRANSFER, 1 );
  curl_setopt( $ch, CURLOPT_FOLLOWLOCATION, 0 );
  curl_setopt( $ch, CURLOPT_SSL_VERIFYPEER, 0 );
  curl_setopt( $ch, CURLOPT_SSL_VERIFYHOST, 0 );

  $timeout = 2; // seconds
  curl_setopt( $ch, CURLOPT_TIMEOUT, $timeout );
  curl_setopt( $ch, CURLOPT_CONNECTTIMEOUT, $timeout );
  
  $contents = curl_exec($ch);
  $ret_code = curl_getinfo( $ch, CURLINFO_HTTP_CODE );
  $errno = curl_errno($ch);
  $error_str = curl_error($ch);

  if( $errno || $error_str ) {
    //print "Error: ${error_str} (${errno})\n";
  }

  //print "Response code: ${ret_code}\n";
  //print "Contents:\n${contents}\n\n";
 
  curl_close($ch);
 
  $data = array(
    'return_code' => $ret_code,
    'contents'    => $contents,
    'error_str'   => $error_str,
    'errno'       => $errno 
  );

  return $data;
}

///////////////////////////////////////////////////////////////////////////////
//  FUNCTION oauth_response_to_array
/// @brief Break up the oauth response data into an associate array
///////////////////////////////////////////////////////////////////////////////
function oauth_response_to_array( $response ) {
  $data = array();
  foreach( explode( '&', $response ) as $param ) {
    $parts = explode( '=', $param );
    if( count( $parts ) == 2 ) {
      $data[urldecode($parts[0])] = urldecode($parts[1]);
    }
  }
  return $data;
}

///////////////////////////////////////////////////////////////////////////////
//  FUNCTION get_request_token
/// @brief Get a request token for a given application.
///////////////////////////////////////////////////////////////////////////////
function get_request_token( $consumer_key, $consumer_secret ) {

  $url = 'https://api.login.yahoo.com/oauth/v2/get_request_token';
  $signature_method = 'plaintext';

  $token = NULL;
  $token_secret = NULL;

  // Add in the lang pref and callback
  $xoauth_lang_pref = 'en-us';
  $oauth_callback = 'oob';  // Set OOB for ease of use -- could be a URL
  
  $params = array( 'xoauth_lang_pref' => $xoauth_lang_pref,
                   'oauth_callback'   => $oauth_callback );

  // Make the signed request without any token
  $response_data = _make_signed_request( $consumer_key, $consumer_secret, $token, $token_secret, $signature_method, $url, $params );

  if( $response_data && $response_data['return_code'] == 200 ) {

    $contents = $response_data['contents'];
    $data = oauth_response_to_array( $contents );

    //print_r( $data );

    return $data;
  }

  return false;
}

///////////////////////////////////////////////////////////////////////////////
//  FUNCTION get_access_token
/// @brief Get an access token for a certain user and a certain application,
///        based on the request token and verifier
///////////////////////////////////////////////////////////////////////////////
function get_access_token( $consumer_key, $consumer_secret, $request_token, $request_token_secret, $verifier ) {

  $url = 'https://api.login.yahoo.com/oauth/v2/get_token';
  $signature_method = 'plaintext';

  // Add in the oauth verifier
  $params = array( 'oauth_verifier' => $verifier );

  // Make the signed request using the request_token data
  $response_data = _make_signed_request( $consumer_key, $consumer_secret, $request_token, $request_token_secret, $signature_method, $url, $params );
  
  if( $response_data && $response_data['return_code'] == 200 ) {

    $contents = $response_data['contents'];
    $data = oauth_response_to_array( $contents );

    //print_r( $data );

    return $data;
  }

  return false;
}


///////////////////////////////////////////////////////////////////////////////
//  FUNCTION make_request
/// @brief Make an actual request to the fantasy API.
///////////////////////////////////////////////////////////////////////////////
function make_request( $consumer_key, $consumer_secret, $access_token, $access_token_secret, $url ) {

  $signature_method = 'hmac-sha1';
  
  // Make the signed request to fantasy API
  $response_data = _make_signed_request( $consumer_key, $consumer_secret, $access_token, $access_token_secret, $signature_method, $url );

  return $response_data;
}


// **** MAIN PROGRAM STARTS HERE ****

$consumer_key = $consumer_data['test']['key'];
$consumer_secret = $consumer_data['test']['secret'];

// 1. Get Request Token
$request_token_data = get_request_token( $consumer_key, $consumer_secret );

if( ! $request_token_data ) {
  print "Could not retrieve request token data\n";
  exit;
}

$request_token = $request_token_data['oauth_token'];
$request_token_secret = $request_token_data['oauth_token_secret'];
$auth_url = $request_token_data['xoauth_request_auth_url'];

// 2. Direct user to Yahoo! for authorization (retrieve verifier)
print "Hey! Go to this URL and tell us the verifier you get at the end.\n";
print ' ' . $auth_url . "\n\n";

print "Type the verifier and hit enter...\n";
$verifier = fgets( STDIN );

print "Here's the verifier you gave us: ${verifier}\n";

// 3. Get Access Token
$access_token_data =
  get_access_token( $consumer_key, $consumer_secret, $request_token, $request_token_secret, $verifier );

if( ! $access_token_data ) {
  print "Could not get access token\n";
  exit;
}

$access_token = $access_token_data['oauth_token'];
$access_token_secret = $access_token_data['oauth_token_secret'];

// 4. Make request using Access Token
$base_url = 'http://fantasysports.yahooapis.com/';
if( isset( $argv[1] ) ) {
  $request_uri = $argv[1];
} else {
  $request_uri = 'fantasy/v2/game/nfl';
}
$request_url = $base_url . $request_uri;

print "Making request for ${request_url}...\n";

$request_data = make_request( $consumer_key, $consumer_secret, $access_token, $access_token_secret, $request_url );

if( ! $request_data ) {
  print "Request failed\n";  
}

$return_code = $request_data['return_code'];
$contents = $request_data['contents'];

print "Return code: ${return_code}\n";
print "Contents:\n${contents}\n\n";

print "Successful\n";

?>
```
        
### PUTs and POSTs

The default PHP OAuth extension does not support any methods aside from GET (as far as I can tell), but we do have write operations as part of the Fantasy API that you might want to make use of. The easiest solution would be to find a library that does support PUTs and POSTs (and DELETEs), but failing that, you can check out this quick sample code that will execute a PUT or POST given an access token (see earlier scripts for how to retrieve an access token).

```
<?php
///////////////////////////////////////////////////////////////////////////////
//  FUNCTION make_write_request
/// @brief Make a write (PUT/POST) request
///
/// @param[out] $auth_failure     Sets variable to true on 401 HTTP code (auth)
/// @param[in]  $consumer_key     Application consumer key
/// @param[in]  $consumer_secret  Application consumer secret
/// @param[in]  $access_token     Access token for user/application
/// @param[in]  $access_secret    Access token secret for user/application
/// @param[in]  $method           PUT or POST
/// @param[in]  $url              URL to PUT/POST against
/// @param[in]  $infile           Filename specifiying data to PUT/POST
///////////////////////////////////////////////////////////////////////////////
function make_write_request( &$auth_failure, $consumer_key, $consumer_secret, $access_token, $access_secret, $method, $url, $infile ) {

  // Make sure we can open the infile
  $in_fh = NULL;
  if( file_exists( $infile ) &&
      $in_fh = fopen( $infile, 'r' ) ) {

    $input_data = '';
    while( $line = fgets( $in_fh ) ) {
      $input_data .= $line;
    }
    
  } else {
    print "Cannot open infile: ${infile}\n";
    return false;
  }

  $auth_failure = false;
  $response_success = false;

  $oauth_consumer_key = $consumer_key;
  $oauth_consumer_secret = $consumer_secret;
  $oauth_token = $access_token;
  $oauth_token_secret = $access_secret;

  $oauth_signature_method = 'HMAC-SHA1';
  $oauth_nonce = rand( 0, 999999 );
  $oauth_timestamp = time();
  $oauth_version = "1.0";

  $params = array(
    'oauth_consumer_key'     => $oauth_consumer_key,
    'oauth_nonce'            => $oauth_nonce,
    'oauth_signature_method' => $oauth_signature_method,
    'oauth_timestamp'        => $oauth_timestamp,
    'oauth_token'            => $oauth_token,
    'oauth_version'          => $oauth_version,
  );

  // Params need to be sorted by key
  ksort( $params, SORT_STRING );

  // Urlencode params and generate param string
  $param_list = array();
  foreach( $params as $key => $value ) {
    $param_list[] = urlencode( $key ) . '=' . urlencode( $value );
  }
  $param_string = join( '&', $param_list );
  
  // Generate base string (needed for SHA1)
  $base_string = urlencode( $method ) . '&' . urlencode( $url ) . '&' .
    urlencode( $param_string );

  // Generate secret and signature
  $secret = urlencode( $consumer_secret ) . '&' . urlencode( $oauth_token_secret );
  $signature = 
    base64_encode( hash_hmac( 'sha1', $base_string, $secret, true ) );

  // Append signature
  $final_url = $url . '?' . $param_string . '&oauth_signature=' . urlencode( $signature );

  // Make the curl call
  $ch = curl_init();
  curl_setopt( $ch, CURLOPT_HTTPHEADER, array( 'Content-type: application/xml' ) );
  if( $method == 'POST' ) { 
    curl_setopt( $ch, CURLOPT_POST, 1 );
    curl_setopt( $ch, CURLOPT_POSTFIELDS, $input_data );
  } else if( $method == 'PUT' ) {
    
    fseek( $in_fh, 0 );
    
    curl_setopt( $ch, CURLOPT_PUT, 1 );
    curl_setopt( $ch, CURLOPT_INFILE, $in_fh );
    curl_setopt( $ch, CURLOPT_INFILESIZE, strlen( $input_data ) );
    
  }
  curl_setopt( $ch, CURLOPT_URL, $final_url );
  curl_setopt( $ch, CURLOPT_RETURNTRANSFER, true );
  
  $ycw_result = curl_exec( $ch );
  $ret_code = curl_getinfo( $ch, CURLINFO_HTTP_CODE );

  fclose( $in_fh );

  curl_close( $ch );

  if( $ret_code == 401 ) {
    $auth_failure = true;
  } else {
    $response_success = true;
  }

  print "Return code: ${ret_code}\n";
  print "Response from API:\n";
  print_r( $ycw_result );

  return $response_success;
}
?>
```
        
### Making Public Requests

Most of the Fantasy API data relies on 3-legged OAuth, as much of the data is specific to a certain Yahoo! user. However, you can use 2-legged OAuth to request purely public data. 2-legged OAuth effectively boils down to making a request without setting an access token through the default PHP OAuth library, or effectively using your consumer key/secret as the token. 