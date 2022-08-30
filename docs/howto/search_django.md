# How to Implement Search Functionality In Your Django app

In this tutorial, I will teach you how to implement a basic search funtionality using Django [Q objects](https://docs.djangoproject.com/en/4.0/topics/db/queries/#complex-lookups-with-q-objects). 

This how-to guide will not be an end-to-end walk through of building a django app from scratch. We would only touch on implementing the search funtionality.

I would recommend checking out my django tutorial on building a [django story app](https://my-portfolio.readthedocs.io/en/latest/tutorial/django_tutorial/) if you need a django project/app to carry out this guide practically for yourself. 

If otherwise you already have a project you are working on and just need to get a search funtionality implemented,then you are good to go.

## Prerequisites & Installations

This guide is not for complete beginners to the Django framework. This guide assume you are comfortable with building a django project/app from scratch; that you are familiar with the django documentation; and that you have moved beyond the django official documentation tutorial to start trying out other parts of the django machinery on your own.

The following prerequisites are also required:

1. Installation of Python3, preferably version 3.8 and above.
2. Installation of Django, preferably version 4 and above.
3. A work-in-progress django project/app (this would be needed to implement the search functionality as I have stated earlier).


!!! info
    I will be using sections from a django project I am currently working on to take you through this guide. You should be able to replicate the functionality in you own project with ease.


## Using Q Objects to Implement a Search Functionality

A basic search funtionality in django can be implemented using the `filter()` method. To search mulitiple fields in your database, you can achieve that by [chaining filters](https://docs.djangoproject.com/en/4.0/topics/db/queries/#chaining-filters).

However, there are instances when it would makes good sense to just simply use Django Q objects, especially when you intend to search multiple fields in your database. It makes for a more efficient and cleaner code. Q objects was built for this purpose. 

Q objects comes with the "OR"  and "AND" logical operators to allow you join several query search lookups for a more robust search functionality.

Moving on, we will start with a section from the `models.py` of my ecommerce project that contains the model whose fields would be needed to implement the search funtionality.

### Step One: Models

In the `models.py`, I have a Product model that handles product items in my ecommerce website.

```
<!---models.py--->

...
from django.db import models
...


class Product(models.Model):
    title = models.CharField(max_length=100)
    original_price = models.DecimalField(max_digits=10, decimal_places=2)
    discount_price = models.DecimalField(max_digits=10, decimal_places=2 , blank=True, null=True)
    product_image = models.ImageField(upload_to= "image/")
    slug = models.SlugField(unique=True)
    is_featured = models.BooleanField(default=False)
    is_bestseller = models.BooleanField(default=False)
    is_new = models.BooleanField(default=False)
    description = models.TextField()

    def __str__(self):
        return self.title

```
I want buyers to be able to search the product page (the page that list all product items) by checking up the `title` and `description` fields of each product item stored in the database. Let's do that.

### Step Two: Views

In the views, I imported the product model; and from django, I imported Q objects.

I would also be using field lookups and the `filter()` method to create each search query lookup.

The view would be a class-based view that would be inheriting from django `ListView`, and I will be over-writing its `get_queryset()` method.

```
<!---views.py--->
...
from django.db.models import Q
from django.views.generic import ListView
from .models import Product
...

class ProductListView(ListView):
    template_name= "product-list.html"
    model = Product  
    
    def get_queryset(self):
        queryset = Product.objects.all()
        query_search = self.request.GET.get('search')

        if query_search:
            queryset = Product.objects.filter(Q(title__icontains=query_search)| Q(description__icontains=query_search)).distinct()

        return queryset



```

In the `get_queryset()` method, we have the `query_search` variable to get the search query entered by the buyer.

If the buyer enters a search query, the `if` block is triggered, and the search query is looked up in the `title` and `description` field of each product item to check if they contain the search query. The filtered result would be returned as a queryset contained in the `queryset` variable.

The `icontains` field lookup is utilized to ensure case insentivity of search results. The `distinct()` method is used to ensure filtered results using Q objects are unique(that is, no duplicates).  

You will notice I used the "OR" logical operator which is singnified by the pipe operator (`|`) for joining the search query lookups. 

### Step Three: Templates

In my `product-list.html`, I added in an `input` interface to enable a buyer enter a search query. 

It is important to specify the `name` attribute of the `input` HTML tag as that is the identifier that allows the `views.py` fetch the search query. The value of the `name` attribute will be `search` same as the argument passed into the `get()` method in the `views.py`.

```
<!--product-list.html-->

{% extends 'base.html' %}



{% block content %}


<section class="text-center mb-4 item">
    <form class="d-flex">
        <input class="form-control me-2" type="search" placeholder="Search" name="search" aria-label="Search" id="item3">
        <button class="btn btn-outline-warning" type="submit">Search</button>
    </form>
    .....
    <!--product-list item code handled here-->
</section>

{% endblock content %}
```

That done, we can now search the product list with search queries.

<figure markdown align="center">
![All Product Items](img/15.png)
  <figcaption><i>All Product Items</i></figcaption>
</figure>

<figure markdown align="center">
![Product item matching search query](img/16.png)
  <figcaption><i>Product item matching search query</i></figcaption>
</figure>

## See Also

So that's it on implementing a basic search fuctionlity with  Q objects in django. 

You can also check out [search lookup functionality](https://docs.djangoproject.com/en/4.0/ref/contrib/postgres/search/) for implementing search in postgreSQL, if you are using the PostgreSQL database for your project.












