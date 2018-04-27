---
layout: post
title: SSO Logout Workaround in OpenStack Horizon
tags: [OpenStack]
---

OpenStack Horizon supports single sign-on login through Keystone. When logging out after authenticating this way, the user should again be sent to keystone, so that keystone too terminates the session correctly. This doesn't happen however.
<!--more-->

There is a setting called `LOGOUT_URL` for doing exactly that, but it has no effect on what happens during logout. I filed [bug 1747149](https://bugs.launchpad.net/horizon/+bug/1747149) describing the issue.

Until that bug is resolved, this post will describe the workaround I'm using. While this is specific to OpenID Connect with `mod_auth_openidc`, it should be the same with the exception of the url to redirect to.

# Logging out of your identity provider
When using `mod_auth_openidc`, you can log out the user by redirecting them to `<IDENTITY_URL>/v3/auth/OS-FEDERATION/websso?logout=<redirect>`. In this case `<redirect>` refers to where the user is sent after a successful logout. It must be url encoded. It must also be a valid `redirect_uri` for your identity provider, because `mod_auth_openidc` will send you there to log out with your identity provider and set `redirect_uri` to `<redirect>`.

So if Keystone is being served at `https://example.com/identity` and Horizon is at `https://example.com/dashboard`, your full logout path will look like this `https://example.com/identity/v3/auth/OS-FEDERATION/websso?logout=https%3A%2F%2Fexample.com%2Fdashboard`.

If you're not using `mod_auth_openidc`, find out the correct logout path, and use that.

# Logging out of Horizon
While the above logs out of your identity provider, it doesn't log you out of Horizon. For that we need to clear the cookies that have been set to identity our user. The specific one is `sessionid`, but there are other's like `login_region` and `login_domain` which should also be clear.

# Putting it all together
We're going to create a html page which is served at the `/dashboard/auth/logout/` path, replacing the current Horizon logout page.

So to summarize: we must unset the cookies, and *then* redirect the user to Keystone's logout path.

Create a new file in `/var/www/html/logout.html` with the below contents, replacing the string with the logout url for your Apache module like described above.

```html
<html>
<head>
<meta http-equiv="refresh" content="2; url=https://example.com/identity/v3/auth/OS-FEDERATION/websso?logout=https%3A%2F%2Fexample.com%2Fdashboard" />
</head>
<body>
<p>Please wait...</p>
</body>
</head>
</html>
```

To have this page served at `/dashboard/auth/logout` we have to edit the Horizon configuration for Apache. Location and exact naming depends on the distribution. For Devstack with Ubuntu it is at `/etc/apache2/sites-available/horizon.conf`. Modify the file to contain the `LoadModule`, `Alias` and `Location` inside of `<VirtualHost>` like below.

```
<VirtualHost>
...

  LoadModule headers_module modules/mod_headers.so
  Alias "/dashboard/auth/logout/" "/var/www/html/logout.html"
  <Location "/dashboard/auth/logout/">
      Header add Set-Cookie "sessionid=; path=/; expires=25 Dec 1991 12:00:00 UTC"
      Header add Set-Cookie "login_domain=; path=/; expires=25 Dec 1991 12:00:00 UTC"
      Header add Set-Cookie "login_region=; path=/; expires=25 Dec 1991 12:00:00 UTC"
  </Location>
 
</VirtualHost>
```

Save and restart Apache.

If everything works correctly, now clicking **Sign Out** in Horizon should log you out of both Horizon and your identity provider.
