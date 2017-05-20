# variants, extending core classes, modifying css using less

<p dir=ltr><-- <a href="/demo/2.payment/README.md">payment (using django-oscar-paypal) and template customization</a></p>
<p dir=rtl> <-- <a href="/demo/4.custom-pricing/README.md">pricing customization</a></p>

Our shop sells lectures and lectures have dates.. So, we should allow each lecture to have multiple dates, and you should be able to buy a ticket for a specific date.  
我們的商店銷售課程，不同的課程有不同的上課日期；所以我們應讓每一種課程可以擁有多個上課日期。並且可以為特定的日期購買上課票。

We will use the oscar variants feature, so that each variant is a date of the same lecture. To do that, we need to delete the existing parnet product stock and add variants for each possible date.  
我們將使用oscar變異化的功能，同一種課程可以有不同的上課日期。為達到此功能，我們得刪除父產品庫存，並且為可能的日期增加變異化。

Add a "date" product attribute
* dashboard > catalogue > product types > edit product type
* product attributes - add date attribute (not required, because otherwise parent will need it as well)    
增加"日期"產品屬性

edit a product and add variants for different dates:
* dashboard > catalogue > products > edit product
* remove existing stock if exists
* add variants + stock for each variant  
編輯產品並增加變異性給不同的日期使用

voila, we have dates

now, let's improve the design of the homepage a bit  
現在讓我們加強一下網頁的設計  
as every django app, let's look at urls to see who handles the homepage url  
一如每個djano app，讓我們檢視urls，以了解如何處理網頁的url  

* [oscardemo/urls.py](oscardemo/urls.py)
  * oscar uses an [Application](https://github.com/django-oscar/django-oscar/blob/1.1.1/src/oscar/core/application.py) object to manage urls and permission, so let's have a look at the default oscar app to see where the homepage is handled  
  * oscar 使用一個`Application` object 來管理urls與權限，所以讓我檢視一個預設的oscar app 來了解網頁如何被處理。
  * `class Application(object):` 具有5個屬性\6個方法，名稱如下
    * `name`
    * `login_url`
    * `hidable_feature_name`
    * `permissions_map` 預設為空字典
    * `default_permissions` 預設為None
    * `def __init__(self, app_name=None, **kwargs):`
    * `def get_urls(self):` 預設返回一個空串列
    * 進行處理urls的動作 Customise URL patterns.
    * `def get_permissions(self, url):` Return a list of permissions for a given URL name
    * `def get_url_decorator(self, pattern):`
    * 
* [oscar/app.py](https://github.com/django-oscar/django-oscar/blob/1.1.1/src/oscar/app.py)
  * [oscar/apps/promotions/app.py](https://github.com/django-oscar/django-oscar/blob/1.1.1/src/oscar/apps/promotions/app.py)
  * `class PromotionsApplication(Application):`oscar apps內的應用的app.py，繼承`Application`，然後對屬性與方法進行`override`, 所以第一個就是`name = 'promotions'`以及其他增加的屬性，然後是`def get_urls(self):`返回的是`return self.post_process_urls(urls)` 進行處理urls的動作。所有的`application` instant最後都在oscar/app.py 中的`class Shop`用掉。
  * [oscar/apps/promotions/views.py](https://github.com/django-oscar/django-oscar/blob/1.1.1/src/oscar/apps/promotions/views.py)
  * 而views.py中的`CBV`也都是在`app.py`呼叫的。


Ok, so, we found the relevant view, now let's override it.

Oscar uses a class overriding technique where instead of directly importing classes / models you use a get_class or get_model command which allows to return customized versions of core apps.

Oscar has a management command for forking a core app and extending it

* ./manage.py oscar_fork_app promotions oscardemo/
  * this created the forked app directory at [oscardemo/promotions](oscardemo/promotions)
* then, we just need to add it to [INSTALLED_APPS](oscardemo/settings/base.py)
  * note we use the get_core_apps command which hooks into the oscar get_class/get_model system

Now, we can add our customized view class:

* [oscardemo/promotions/views.py](oscardemo/promotions/views.py)
* [templates/promotions/home.html](templates/promotions/home.html)
  * note, that we have bootstrap 3

Let's add some style for our homepage,

Oscar has a command to copy static files locally so you could modify them:

* ./manage.py oscar_fork_statics oscardemo/static
* now all the static files are at [oscardemo/static](oscardemo/static)
* django static file finder looks for static directory inside each installed app, so let's add oscardemo to the [INSTALLED_APPS](oscardemo/settings/base.py)

Oscar uses [Django Compressor](https://django-compressor.readthedocs.org/en/latest/) for less compilation to css. We will see how it works on production later, but this is how it can be setup for development:

* add USE_LESS = True it to the settings override file [oscardemo/settings/override.py.dist](oscardemo/settings/override.py.dist)
  * this setting is passed-on to the context, and checked where css files are included, e.g.: [templates/oscar/layout.html](https://github.com/django-oscar/django-oscar/blob/1.1.1/src/oscar/templates/oscar/layout.html)
* ensure you have lessc binary, as best-practive and to ensure everyone has the same less compiler:
  * I suggest to use nvm - add an [.nvmrc](.nvmrc) file with the required node version
  * on the command line - write "nvm use" to ensure you use this version
* now, start a node project and install less in that project
  * npm init
  * npm install less --save
* now, we use the django-compressor precompilers directive to compile less to css in run-time:
  * [oscardemo/settings/override.py.dist](oscardemo/settings/override.py.dist)

Finally, we are ready to add our custom style

* [oscardemo/static/oscar/less/styles.less](oscardemo/static/oscar/less/styles.less)
* [oscardemo/static/oscar/less/oscardemo/main.less](oscardemo/static/oscar/less/oscardemo/main.less)

Beautiful:

* http://localhost:8000/

<p dir=ltr><-- <a href="/demo/2.payment/README.md">payment (using django-oscar-paypal) and template customization</a></p>
<p dir=rtl> <-- <a href="/demo/4.custom-pricing/README.md">pricing customization</a></p>
