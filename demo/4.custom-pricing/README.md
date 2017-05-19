# pricing customization

<p dir=ltr><-- <a href="/demo/3.variants-and-dates/README.md">variants, extending core classes, modifying css using less</a></p>

Now, let's simplify things for the shop admin:
現在，要簡化商店管理的事項

* The need to input stock records, choosing partner and sku is cumbersome
* 輸入庫存紀錄所需要選擇夥伴與sku顯得多餘的。
  * let's assume we only have 1 partner (ourselves)
  * in that case the sku can be generated automatically
* regarding product price
* 關於產品價格
  * we just want to input the cost price and have the retail price and tax calculated automatically
  * 我們只想要輸入成本價格，讓零售價格與稅金自動算出
Let's start with the dashboard -
從儀表板開始

* we want just to have a cost price and available stock fields available as part of product detail (not in stockrecord)
我們想要讓成本價與可得庫存欄位是產品細節的一部份。

The dashboard uses html templates, so let's find the product edit template
儀表板使用html模板，所以找產品編輯模板出來
* [oscar/dashboard/catalogue/product_update.html](https://github.com/django-oscar/django-oscar/blob/1.1.1/src/oscar/templates/oscar/dashboard/catalogue/product_update.html#L97)
  * ok, it's a standard django form, let's find the form
  * 好的，這是一個標準的django form，讓我們找到表單
* [oscar/apps/dashboard/catalogue/forms.py](https://github.com/django-oscar/django-oscar/blob/1.1.1/src/oscar/apps/dashboard/catalogue/forms.py#L210)

ok, so we should extend the form to add the price field and num in stock field
好的，所以我們應該擴展表單以加入價格欄位及庫存數量(`num_in_stock`)欄位
```bash
$ ./manage.py oscar_fork_app dashboard.catalogue oscardemo/
$ ./manage.py oscar_fork_app catalogue oscardemo/
```

* add it to settings

make the modifications:

* [oscardemo/catalogue/models.py](oscardemo/catalogue/models.py)
* Product繼承AbstractProduct，但不直接增加欄位，而是增加`def cost_price`這直接自`stockrecord`取出`cost_price`
* 再增加`def num_in_stock`這直接自`stockrecord`取出`num_in_stock`
* 再增加`def set_stockrecord(self, cost_price, num_in_stock):` 這直接進行設值給`stockrecord`

* [oscardemo/dashboard/catalogue/forms.py](oscardemo/dashboard/catalogue/forms.py)
```python
# here we override the core oscar product form
# and add the cost_price and num_in_stock fields
class ProductForm(OscarProductForm):

# we remove the currency and price fields as we will only use the cost_price field
class StockRecordFormSet(OscarStockRecordFormSet):

```
* [templates/dashboard/catalogue/product_update.html](templates/dashboard/catalogue/product_update.html)
```html
      <!--<td>{% include "dashboard/partials/form_field.html" with field=stockrecord_form.price_currency nolabel=True %}</td>-->
      <td>{% include "dashboard/partials/form_field.html" with field=stockrecord_form.cost_price nolabel=True %}</td>
      <!--<td>{% include "dashboard/partials/form_field.html" with field=stockrecord_form.price_excl_tax nolabel=True %}</td>-->
      <!--<td>{% include "dashboard/partials/form_field.html" with field=stockrecord_form.price_retail nolabel=True %}</td>-->
      <td>
          {% include "dashboard/partials/form_field.html" with field=stockrecord_form.id nolabel=True %}
          {% include "dashboard/partials/form_field.html" with field=stockrecord_form.DELETE nolabel=True %}
      </td>
```
ok, now dashboard allows to input only cost price and number of units, but if we look at the product on the site, it doesn't have a price
now comes an interesting part of oscar - the product availability and pricing is managing in strategy objects, this allows great flexibility
for our demo, we need to modify the strategy object to calculate the prcies from the cost price
the oscar partner app manages the strategy, so let's fork it:

現在儀表板可以輸入成本價與單位數量，但假如我檢視在網站上的產品，它仍未有價格

```bash
$ ./manage.py oscar_fork_app partner oscardemo/
```

* don't forget to add it to settings..

now, let's see the modifications:

* [oscardemo/partner/strategy.py](oscardemo/partner/strategy.py)
```python
# the selector class allows to change the pricing/availability strategy
class Selector(OscarSelector):

    def strategy(self, request=None, user=None, **kwargs):
        # here we can change the strategy based on request/user etc..
        # but for now we will use the same strategy for all cases
return OscarDemoStrategy(request)

# the strategy object usese multiple classes, each providing some of the functionality
class OscarDemoStrategy(
    UseFirstStockRecord,  # oscar allows multiple stockrecords for the same product
                          # this allows, for example, to have orders of a large quantity be made via a different supplier
                          # but for most cases, we just use the first stock record
    StockRequired,  # this is the basic availability policy strategy
                    # it ensures there is enough stock to allow purchasing the product
                    # also, in case of parent products - if no variants are available, the parent will not be available
    Structured,  # this is the main standard strategy class, providing a lot of common functionality
                 # let's have a look at it: https://github.com/django-oscar/django-oscar/blob/1.1.1/src/oscar/apps/partner/strategy.py#L101
                 # it defines the 2 main functions which any strategy requires -
                 # fetch_for_product and fetch_for_parent (product)
                 # it then returns a PurchaseInfo object containing the price and availability
):

```

* [oscardemo/partner/prices.py](oscardemo/partner/prices.py)
```python
from oscar.apps.partner.prices import FixedPrice
from decimal import Decimal as D


class CostBasedPrice(FixedPrice):

    def __init__(self, currency, cost_price):
        excl_tax = cost_price * D(2)
        tax = excl_tax * D(.18)
super(CostBasedPrice, self).__init__(currency, excl_tax, tax)
```
That's it, simple huh? Now, have a look at the site..

<p dir=ltr><-- <a href="/demo/3.variants-and-dates/README.md">variants, extending core classes, modifying css using less</a></p>
