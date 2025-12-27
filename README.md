# Overview
A Django shopping cart app with an asynchronous, responsive, minimalist, Bootstrap styled template. Supports both ```AnonymousUser``` sessions and authenticated ```User``` sessions. Easily adjusted to account for varying ```Product``` models.

## Requirements
- Django 3.x <!-- tested with 3.1.4 -->
- Bootstrap 4.x <!-- tested with 4.5.0 -->
- jQuery 3.x minified <!-- tested with 3.6.0 -->

## Usage

### Persistent Carts
To allow persistent carts for authenticated users, one must sync cart states of the user session and database at login, as shown below. This can require merging the two carts if both the anonymous session cart and database cart associated with the authenticating user are populated. All of this is handled by the ```sync_session_and_db(request)``` function.

```
class LoginView(auth_views.LoginView):
	def get_success_url(self):
		self.__load_user_data(self.request)
		return super().get_success_url()
  
	def __load_user_data(self, request):
		sync_session_and_db_cart(request)
```

### Using the Provided Forms
One can populate ```AddToCartForm``` in the manner shown below.

```
form = AddToCartForm() 

products = Product.objects.filter(product_name="...")
product_choices = [(i, str(product.quantity)+" items - $"+str(product.product_price)) \  
					for (product, i) in zip(products, range(len(products)))]  
  
form.fields["product_choices"].choices = product_choices
```

Form data can be provided to the user via an ```HttpResponse``` context dictionary.

### Handling Product Page POST Requests
As shown below, product page POST requests are handled with the ```add_to_cart(request, product_id, quantity, reflect_to_db)``` function.

```
products = Product.objects.filter(product_name="...")
if "product_choices" in request.POST and "item_quantity" in request.POST:  
	if request.POST["item_quantity"] != '' \  
			and int(request.POST["item_quantity"]) > 0 \  
			and int(request.POST["item_quantity"]) <= 1000:  
		
		quantity = int(request.POST["item_quantity"])  
		product_id = products[int(request.POST["product_choices"])].product_id  
		add_to_cart(request, product_id, quantity, True)
else:  
	raise Http404("Request missing necessary information!")
```
