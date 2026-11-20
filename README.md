# MageBooster_OdealHyva – Ödeal in MageBooster Checkout (Hyvä and Luma)

Free, MIT. Adds the Ödeal method panel to the **MageBooster Checkout** (Alpine.js one-page checkout), which runs on
Hyvä and Luma themes. Needs `magebooster/module-odeal` (payment, callbacks, refunds stay there).

| Storefront | What renders the method |
|---|---|
| Luma theme, Magento checkout | Knockout renderer of MageBooster_Odeal (this package not needed) |
| Hyvä theme with the Luma checkout fallback | the same Knockout renderer |
| Hyvä or Luma theme + MageBooster Checkout | this package (panel) |
| Hyvä Checkout (Magewire) | not supported yet |

## Install

```
composer require magebooster/module-odeal-hyva
bin/magento module:enable MageBooster_OdealHyva
bin/magento setup:upgrade
```

The blocks go into MageBooster Checkout's `mb.checkout.payment.panels`
containers; without MageBooster Checkout nothing is rendered. They are shown only while the method is active.

## Why this package is small

Ödeal only offers a hosted payment link: the card is always entered on the Ödeal page, so there is no card form
to render and no script. The redirect after "Place order" needs no code here either: `MageBooster_Odeal` sets
`payment/magebooster_odeal/order_place_redirect_url` to `mbodeal/redirect/start`, and MageBooster Checkout's
`mbcheckout/place/next` sends every method that has this value there. The package only adds the method panel
(description, test mode note, TRY amount and rate, "you will be taken to Ödeal" note).

## License

MIT, see `LICENSE`.

---

# MageBooster_OdealHyva – MageBooster Checkout'ta Ödeal (Hyvä ve Luma)

Ücretsiz, MIT. **MageBooster Checkout**'a (Hyvä ve Luma temalarında çalışan Alpine.js tek sayfa ödeme) Ödeal yöntem
panelini ekler: açıklama, test modu notu, TL tutarı ve kuru, Ödeal sayfasına yönlendirme notu.
`magebooster/module-odeal` gerekir (ödeme, dönüş, iade orada). Luma checkout'ta ve Luma checkout'a düşen Hyvä
temasında bu pakete gerek yoktur; Hyvä Checkout (Magewire) henüz desteklenmiyor.
