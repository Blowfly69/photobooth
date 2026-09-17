# Remote PHP debugging

This repository includes a shared VS Code setup for debugging PHP requests on a remote web server with Xdebug 3 and the [PHP Debug](https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug) extension.

## Workstation setup

1. Install the PHP Debug extension in VS Code.
2. Open the repository locally.
3. Set `photobooth.remotePath` in `.vscode/settings.json` to the absolute path of the project on the server. The default is `/var/www/photobooth`.
4. Start **Run and Debug > Listen for Xdebug**.
5. Start the **Open reverse Xdebug SSH tunnel** task and enter the SSH target used to administer the server.

The task runs this tunnel:

```text
ssh -N -R 9003:127.0.0.1:9003 user@example.com
```

Keep that terminal running while debugging. The reverse tunnel lets Xdebug connect to `127.0.0.1:9003` on the server while the connection is delivered to VS Code on the workstation. Port 9003 does not need to be opened in the server firewall.

## Server setup

Install Xdebug for the PHP SAPI used by the web server. On Debian or Ubuntu, for example:

```sh
sudo apt install php8.4-xdebug
```

Add an Xdebug configuration to the web SAPI's additional INI directory, such as `/etc/php/8.4/apache2/conf.d/99-xdebug.ini` or `/etc/php/8.4/fpm/conf.d/99-xdebug.ini`:

```ini
zend_extension=xdebug
xdebug.mode=debug
xdebug.start_with_request=trigger
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
xdebug.idekey=VSCODE
```

Restart the relevant service after changing the INI file:

```sh
sudo systemctl restart apache2
# Or, for PHP-FPM behind Nginx:
sudo systemctl restart php8.4-fpm
```

Use a debugger trigger for a request instead of enabling every request. For example, with a browser extension, set the Xdebug trigger cookie or header to `1`; for a command-line request, use:

```sh
curl -H 'XDEBUG_TRIGGER: 1' https://photobooth.example.com/
```

When a breakpoint is hit, VS Code maps the server files back to this checkout using `photobooth.remotePath`.

## Verify the connection

On the server, check the active web SAPI configuration:

```sh
php --ini
php -m | grep -i xdebug
```

For Apache or PHP-FPM, confirm Xdebug is loaded by a temporary `phpinfo()` page or the existing server diagnostics, then remove any temporary diagnostic file. Do not leave `xdebug.start_with_request=yes` enabled on a public server.

Keep the SSH tunnel and Xdebug listener restricted to administrative access. Do not expose port 9003 to the internet, and disable Xdebug again when remote debugging is finished.
