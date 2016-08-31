
App opens 

check if there is an access_token?

post to /v1/numbers
  find_or_create_number
  cancel other confirmations sent to this phone number?
  create a confirmation
  send a text
  
user gets the text and inputs code

post to /v1/confirmations with number and code
  confirmation is confirmed
  find or create a user for the confirmed phone number
  access_token is created and returned and stored
  
  
user has an access_token and is "logged in"

adding contacts
adding locations


SecureRandom.hex(32)


  
user
  number_id
  confirmation_id
  home_id
  locations
  contacts      
  
