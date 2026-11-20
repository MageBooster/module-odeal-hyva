# MageBooster Ödeal payment: how-to

- **Package:** `magebooster/module-odeal`
- **Licence:** paid (serial number required)
- **Payment method code:** `magebooster_odeal`
- **Support:** info@magebooster.com

## What it does

This module adds Ödeal as a card payment method. It uses Ödeal's hosted 3D Secure payment page.

### Payment flow

1. The customer places the order. The order is saved as **Pending Payment**, and the order email waits until the payment is confirmed.
2. The module gets an access token from Ödeal (`/token`).
3. It creates a payment page (`/vpos/pay-by-link`) with the order number as `externalId`, then sends the customer to that page.
4. The customer enters the card on Ödeal's page and completes 3D Secure.
5. The customer returns to `/mbodeal/payment/callback`. The module asks Ödeal for the result (`/vpos/check-status`).
6. The order is updated:
   - **Paid:** an invoice is created and the order moves to **Processing**.
   - **Failed or cancelled:** the order is cancelled and the cart is restored.

### Other behaviour

- **Missed returns:** the cron job `magebooster_odeal_reconcile` runs every 10 minutes. It checks orders whose customer closed the browser before coming back.
- **Unpaid orders:** these are cancelled by Magento's own **Pending Payment Order Lifetime** setting (**Stores > Configuration > Sales > Sales > Orders Cron Settings**).
- **Refunds:** an online credit memo calls the refund path set in the admin.
- **Return URLs:** they carry the order number signed with your store's crypt key.
- **Card details:** customers enter them on Ödeal's page only. Your store never sees them.
- **Currency:** an order in a currency outside "Currencies charged directly" is converted to TRY with the rate saved in **Stores > Currency Rates**. The TRY amount and rate appear at checkout and on the order.

## Requirements

- PHP 8.0 or later
- Magento 2.4.4 to 2.4.9
- An Ödeal merchant account with API access. Ödeal provides the Client ID, the client secret and the test and live API addresses.
- Luma edition: `magebooster/module-odeal`
- Hyvä edition: `magebooster/module-odeal-hyva`, a separate download

### Hyvä status

This package contains a Knockout payment renderer for Luma-based checkouts. The server side works the same on any theme. The module's README lists the Hyvä Checkout component as planned. Confirm that the Hyvä edition is available before you use it on a Hyvä store.

## Install with Composer

```bash
composer require magebooster/module-odeal
```

## Install from a zip file

Unzip the archive into `app/code/MageBooster/Odeal`, then run the commands below.

## Enable the module

```bash
bin/magento module:enable MageBooster_Odeal
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento setup:static-content:deploy -f
bin/magento cache:flush
```

Magento cron must be running so that the reconcile job runs.

## Admin configuration

Go to **Stores > Configuration > Sales > Payment Methods > Ödeal · MageBooster**.

| Field | Default | What it means |
|---|---|---|
| Enabled | No | Shows the method at checkout. |
| Title | Credit / debit card (Ödeal) | Name at checkout and on the order. |
| Text shown at checkout | (empty) | Short text under the method name. |
| Test mode | Yes | Uses the test API address and test credentials. No real money is taken. |
| Client ID | (empty) | Given by Ödeal. |
| Client secret | (empty) | Given by Ödeal. Stored encrypted. |
| Grant type | client_credentials | As given by Ödeal. |
| Scope | (empty) | As given by Ödeal. |
| Token path | /token | Path used to get the access token. |
| Refund path | /vpos/refund | Path used for refunds. Confirm it with Ödeal. |
| Test API address | (empty) | Ödeal gives it together with the test credentials. |
| Live API address | (empty) | Ödeal gives it together with the live credentials. |
| Currencies charged directly | TRY | Comma-separated list. Add other currencies only if Ödeal enables them for you. |
| Payment from applicable countries | All allowed countries | Limits the method by country. |
| Payment from specific countries | (empty) | Used when specific countries are chosen. |
| Sort order | 62 | Position among the payment methods. |
| Debug log | No | Writes requests and answers to `var/log/magebooster_mbodeal.log`. Card data and secrets are never written. |

## Testing

1. Turn on **Test mode** and **Debug log**, then enter the test credentials and test API address from Ödeal.
2. Place an order and pay on the Ödeal page with a test card. The order should become **Processing** and have an invoice.
3. Start another order and close the browser on the Ödeal page.
   - After paying there, wait up to 10 minutes. The cron job should update the order.
   - If you do not pay, the order is cancelled by Pending Payment Order Lifetime.
4. Cancel on the Ödeal page. The order should be cancelled and the cart restored.
5. Create an online credit memo for a paid order. Check the log for the refund call.
6. Switch off Test mode only after these steps pass.

## Known limits

- **Not yet tested against Ödeal.** No test credentials were available while building this module.
- **Undocumented API details.** Ödeal's public docs do not give the API addresses, the token answer fields or the refund endpoint, so they are admin settings. For the token, the module accepts `accessToken`, `access_token` or `token`.
- **One token at a time.** Each new token invalidates the previous one. If many payments start in the same second, one of them may need a retry.

## Troubleshooting

- **The method does not show at checkout.** Check **Enabled**, the API addresses, the country settings and the currency list.
- **Token error in the log.** Check the Client ID, secret, grant type, scope and token path with Ödeal.
- **The order stays in Pending Payment.** Check that cron is running and that `/mbodeal/payment/callback` is reachable from outside.

## Uninstall

1. Turn off **Enabled**.
2. Wait until no orders are in Pending Payment.
3. Run:

   ```bash
   bin/magento module:disable MageBooster_Odeal
   composer remove magebooster/module-odeal
   bin/magento setup:upgrade
   bin/magento cache:flush
   ```

Ödeal is a trademark of its owner. This module is not made or endorsed by Ödeal.

## Licence & copying

Commercial licence, see `LICENSE.txt`. Copyright (c) 2026 WebWorks 101 OÜ (MageBooster). All rights reserved.

- One serial number = one production installation (one store domain). Staging and development copies of that same store are allowed.
- Each serial number and each download is tied to the customer who bought it. Luma and Hyvä editions are separate products with separate serial numbers.
- You may modify the code for your own licensed store. Keep the copyright and licence notices.
- Not allowed: copying to other stores, redistributing, reselling, sublicensing, publishing in public repositories, marketplaces or forums, removing licence checks or these notices, sharing serial numbers.
- In case of unauthorised copying or use, the serial number is revoked without refund, updates and support stop, and WebWorks 101 OÜ may claim damages and take legal action. Estonian law applies; the court is Harju County Court.

## Support

info@magebooster.com
