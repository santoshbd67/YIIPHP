How to fix it:
You need to set a value for cookieValidationKey in your config/web.php file.

Find your config/web.php (or similar config file), and inside the components section, add or update the request component like this:

php
Copy
Edit
'components' => [
    'request' => [
        // !!! insert a secret key here (this is required by cookie validation)
        'cookieValidationKey' => 'your-very-secret-random-string',
    ],
    // other components...
],
Replace 'your-very-secret-random-string' with a long, random string.
You can generate a key easily:

To generate a random key:

bash
Copy
Edit
php -r "echo bin2hex(random_bytes(32));"
This will output a secure random key like:

Copy
Edit
4f2c56e1a8b5f0b5a6a8827db23bb370ca2ef0d728cf0340a2c0ee8127edc135
Use that value in cookieValidationKey.

Example after fix:
php
Copy
Edit
'request' => [
    'cookieValidationKey' => '4f2c56e1a8b5f0b5a6a8827db23bb370ca2ef0d728cf0340a2c0ee8127edc135',
],
✅ After doing this, rebuild your Docker image and restart the container if needed.
