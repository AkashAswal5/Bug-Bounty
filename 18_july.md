create multiple user

admin@admin.com
Password123!


```
php aritsan thinker
use BookStack\Users\Models\User;
User::all();

    or

User::select('id', 'name', 'email', 'slug')->get();
```


| Name        | Username / Slug | Email                 | Password         | Assigned Role | Purpose                                                                           |
| ----------- | --------------- | --------------------- | ---------------- | ------------- | --------------------------------------------------------------------------------- |
| Admin   | `admin`         | `admin@admin.com`     | `Password123!`   | Admin     | Full administrative access. Manage users, roles, permissions, and settings.       |
| Guest   | `guest`         | `guest@example.com`   | *(No login)*     | Public    | Represents anonymous visitors accessing public content.                           |
| Alice   | `alice`         | `alice@example.com`   | `Password123!`   | Editor    | Owner of **Book A**. Used to test ownership and authorization.                    |
| Bob     | `bob`           | `bob@example.com`     | `Password123!`   | Editor    | Owner of **Book B**. Used for cross-user authorization testing.                   |
| Charlie | `charlie`       | `charlie@example.com` | `Password123!`   | Viewer    | Read-only user. Tests view-only permissions.                                      |
| Dave    | `dave`          | `dave@example.com`    | `Password123!`   | Viewer    | Second read-only user. Useful for comparing viewer behavior.                      |
| Eve     | `eve`           | `eve@example.com`     | `Password123!`   | Editor    | Additional editor for testing collaboration and permission inheritance.           |
| Mallory | `mallory`       | `mallory@example.com` | `Password123!`   | Editor    | "Attacker" account used to attempt unauthorized access to other users' resources. |


| User    | Content to Create                                                  |
| ------- | ------------------------------------------------------------------ |
| Alice   | Book A → Chapter A1 → Page A1 → Upload `alice.pdf` and `alice.png` |
| Bob     | Book B → Chapter B1 → Page B1 → Upload `bob.pdf` and `bob.png`     |
| Eve     | Book C → Private notes and attachments                             |
| Charlie | No content (view-only account)                                     |
| Dave    | No content (view-only account)                                     |
| Mallory | Test content used as the attacker account                          |


create a new role: `private editor` for eve so --> eve an read, write, delete its own assets, no one else except (admin)

#### setup for testing:
Authentication: Login, logout, password reset, session management.
Authorization (IDOR/BOLA): Can Bob modify Alice's pages? Can Mallory access Eve's private book?
Privilege escalation: Can a Viewer perform Editor actions?
Resource permissions: Public vs private books, page restrictions, attachment access.
Business logic: Sharing, ownership transfer, deletion, restoration, exports.
API security: Verify that API endpoints enforce the same permissions as the web interface.
    
