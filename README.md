Environment setup instructions for Map Admin Tool
Any environment that can run Django
Updated 24/02/2020

Tested on Python2.7 and Python3.7.

## Install Virtualenv and Virtualenvwrapper ##

    for Python2 or Python3 

## Install Django ##

    for python2.7

        pip install -r requirements_2.txt

    for python3.7

        pip install -r requirements_3.txt

## Include the app in settings.py ##

    INSTALLED_APPS = [
        ...,
        'gis_map_view',
        ...
    ]

## ================================== ##
## Loading gis_map_view into your app ##
## ================================== ##

## Add it to the urls ##

    > Note: you can use the url path of your choice e.g. ^map_view/

    > Do not change the views name nor the url pattern name 'gis_tool' - it is used in the js
    
    urlpatterns = [
        ...,
        url(r'^$', views.gis_tool, name="gis_tool"),
        ...,
    ]

## Create and save an .env file in the root of your Django project ##

    > Assign values to the following variables

    TOKEN = 'Token xxxxxxxxxxxxxxxxxxxxxxxxxx'
    API_KEY = '++++++++++++++++++++++++++++++'

    # header snippet url which configures how the map looks
    CAMPAIGN_URL = 'https://app.m4jam.com/&&&&&&&&&&&&&&&&&&&?file=json'
    # payloaad snippet url
    PAYLOAD_URL = 'https://app.m4jam.com/********************?file=json'


## In views file ##

    ...
    from django.http import JsonResponse
    from decouple import config
    import json
    ...

    def gis_tool(request):

        # parameters for our geojson request
        # API_KEY and TOKEN are in .env 
        
        geojson_call = {
            'API_KEY': config('API_KEY', default=''),
            'TOKEN': config('TOKEN', default=''),
            'PAYLOAD_URL': config('PAYLOAD_URL', default=''),
            'CAMPAIGN_URL': config('CAMPAIGN_URL', default=''),
            'request': request
        }

        context = { 
            ...,
            'geojson_call': geojson_call,
            ...,
        }

        # checks whether indexeddb has data, if not, refreshes and gets data from snippet 
        
        if request.is_ajax():
            
            if request.method == 'POST':
                
                if json.loads(request.body)['request'] == 'refresh':

                    if 'snippet_result' in request.session:
                        del request.session['snippet_result']
                    if 'snippet_success' in request.session:
                        del request.session['snippet_success']

                    return JsonResponse({ "validation_response": '' })
            

        return render(request, 'index.html', context)

## In the htnl file used to view the map e.g. index.html ##

    {% load staticfiles %}
    {% load gis_map %}
    ...

    <!doctype html>
    <html>
        <head>
            ...
            <!-- initialse static files from the gis app -->
            {% block gis_head %}

                {{ geojson_call | gis_setup }}
                    
            {% endblock gis_head %}
            ...
        </head>

        <body class="text-left">
            
            {% csrf_token %}
            
            <!-- body from the leaflet app -->
            {% block gis_body %}

                {{ geojson_call | gis_view }}

            {% endblock gis_body %}
        </body>
    </html>


## Running the tool ##
    
    ./manage.py makemigrations
    ./manage.py migrate
    ./manage.py collectstatic
    ./manage.py runserver

    > open the url in any browser (preferably Chrome)
