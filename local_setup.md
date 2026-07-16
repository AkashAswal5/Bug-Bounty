if we pull new changes

```
git pull
composer install
php artisan migrate
php artisan optimize:clear
php artisan serve
```


If you restart your PC

```
sudo systemctl start mariadb
cd ~/Bug-bounty-projects/bookstack
php artisan serve
```

Check if any users exist
```
php artisan tinker
BookStack\Users\Models\User::count();
```

Let's see who those users are
` BookStack\Users\Models\User::select('id','name','email')->get(); `

Reset the password
``` 
$user = BookStack\Users\Models\User::where('email', 'admin@admin.com')->first();
$user->password = bcrypt('Password123!');
$user->save();
```

> docu: create a admin user:  https://www.bookstackapp.com/docs/admin/commands/#create-an-admin-user

` php artisan bookstack:create-admin `

