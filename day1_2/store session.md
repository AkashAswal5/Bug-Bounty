discourse.com --> ruby on rails

cookie format
	Set-Cookie:
		_forum_session=abc123.....

----
##### notes
> can use cookie after we log out  --> NO --> 
> can

test for 
>Http only, secure
> try BOLA, interchange cookie
> created multiple account + 1 admin and test for admin session to bypass 

> create multiple user at same time --> but it require human interaction, like to click link for activate account
> send multiple request  on  up to 15 https://try.discourse.org/u/testing_hacking.json user profile to get date [race condition] , it doesn't work

> try to delete other user session https://try.discourse.org/session/testing_hacking1 not work

