# The Movie DB Addon
## This Add-on allows you to ingest Movie DB data from their API.

Input Name: movie_db_upcoming_insights
Sourcetype: movie-db:insights:upcoming
Description: Get Insights from Movie DB database.
Interval: 3600

## Configuration:

### Base URL (base_url)
* Placeholder: api.themoviedb.org/3/movie/
* Ideally, we would want this to be inside the code, but for flexibility, we are allowing end-user to modify the base URL.

### API Key (api_key)

* Placeholder: Enter your API key here.
* Description: Saves API key in Splunk Secret Storage endpoint.
* Type: Secret Password

## Data Input 

### SSL Verify (ssl_verify)
* Type: checkbox
* 


```py

# encoding = utf-8

import os
import sys
import time
import datetime

import requests as req, json

'''
    IMPORTANT
    Edit only the validate_input and collect_events functions.
    Do not edit any other part in this file.
    This file is generated only once when creating the modular input.
'''
'''
# For advanced users, if you want to create single instance mod input, uncomment this method.
def use_single_instance_mode():
    return True
'''

def validate_input(helper, definition):
    """Implement your own validation logic to validate the input stanza configurations"""
    # This example accesses the modular input variable
    # ssl_verify = definition.parameters.get('ssl_verify', None)
    pass

def tmdb_api_call(requestURL, parameters):
    response = req.get(url=requestURL, params=parameters)
    if response.status_code != 200:
        print(response.json())
        exit()
    data = response.json()
    return data

def get_upcoming_movie(base_url, api_key, page_num=1):
    requestURL = f"https://{base_url}/upcoming"
    parameters = {
        "api_key": api_key,
        "page": page_num
    }

    return tmdb_api_call(requestURL, parameters)

def write_in_splunk(data, helper, ew, sourcetype=helper.get_sourcetype()):
    event = helper.new_event(source=helper.get_input_type(), index=helper.get_output_index(), sourcetype=sourcetype, data=data)
    ew.write_event(event)
    
def collect_events(helper, ew):
    
    
    
    opt_ssl_verify = helper.get_arg('ssl_verify')
    api_key = helper.get_global_setting("api_key")
    base_url = helper.get_global_setting("base_url")
    
    upcoming_movies = get_upcoming_movie(base_url, api_key, 1)
    try:
        
        data = upcoming_movies.get('results',[])
    
        for movie in data:
            movie["poster_path"] = "https://www.themoviedb.org/t/p/w440_and_h660_face/" + movie["poster_path"]
            movie["backdrop_path"] = "https://www.themoviedb.org/t/p/w440_and_h660_face/" + movie["backdrop_path"]
        
            write_in_splunk(json.dumps(movie), helper, ew)
    except Exception as e:
        write_in_splunk(e, helper, ew, sourcetype="movie_db_upcoming_insights:error")
    #for movie in data["results"]:
    #    write_in_splunk(movie, helper, ew)
    
    
    
    
    """Implement your data collection logic here

    # The following examples get the arguments of this input.
    # Note, for single instance mod input, args will be returned as a dict.
    # For multi instance mod input, args will be returned as a single value.
    opt_ssl_verify = helper.get_arg('ssl_verify')
    # In single instance mode, to get arguments of a particular input, use
    opt_ssl_verify = helper.get_arg('ssl_verify', stanza_name)

    # get input type
    helper.get_input_type()

    # The following examples get input stanzas.
    # get all detailed input stanzas
    helper.get_input_stanza()
    # get specific input stanza with stanza name
    helper.get_input_stanza(stanza_name)
    # get all stanza names
    helper.get_input_stanza_names()

    # The following examples get options from setup page configuration.
    # get the loglevel from the setup page
    loglevel = helper.get_log_level()
    # get proxy setting configuration
    proxy_settings = helper.get_proxy()
    # get account credentials as dictionary
    account = helper.get_user_credential_by_username("username")
    account = helper.get_user_credential_by_id("account id")
    # get global variable configuration
    global_userdefined_global_var = helper.get_global_setting("userdefined_global_var")

    # The following examples show usage of logging related helper functions.
    # write to the log for this modular input using configured global log level or INFO as default
    helper.log("log message")
    # write to the log using specified log level
    helper.log_debug("log message")
    helper.log_info("log message")
    helper.log_warning("log message")
    helper.log_error("log message")
    helper.log_critical("log message")
    # set the log level for this modular input
    # (log_level can be "debug", "info", "warning", "error" or "critical", case insensitive)
    helper.set_log_level(log_level)

    # The following examples send rest requests to some endpoint.
    response = helper.send_http_request(url, method, parameters=None, payload=None,
                                        headers=None, cookies=None, verify=True, cert=None,
                                        timeout=None, use_proxy=True)
    # get the response headers
    r_headers = response.headers
    # get the response body as text
    r_text = response.text
    # get response body as json. If the body text is not a json string, raise a ValueError
    r_json = response.json()
    # get response cookies
    r_cookies = response.cookies
    # get redirect history
    historical_responses = response.history
    # get response status code
    r_status = response.status_code
    # check the response status, if the status is not sucessful, raise requests.HTTPError
    response.raise_for_status()

    # The following examples show usage of check pointing related helper functions.
    # save checkpoint
    helper.save_check_point(key, state)
    # delete checkpoint
    helper.delete_check_point(key)
    # get checkpoint
    state = helper.get_check_point(key)

    # To create a splunk event
    helper.new_event(data, time=None, host=None, index=None, source=None, sourcetype=None, done=True, unbroken=True)
    """

    '''
    # The following example writes a random number as an event. (Multi Instance Mode)
    # Use this code template by default.
    import random
    data = str(random.randint(0,100))
    event = helper.new_event(source=helper.get_input_type(), index=helper.get_output_index(), sourcetype=helper.get_sourcetype(), data=data)
    ew.write_event(event)
    '''

    '''
    # The following example writes a random number as an event for each input config. (Single Instance Mode)
    # For advanced users, if you want to create single instance mod input, please use this code template.
    # Also, you need to uncomment use_single_instance_mode() above.
    import random
    input_type = helper.get_input_type()
    for stanza_name in helper.get_input_stanza_names():
        data = str(random.randint(0,100))
        event = helper.new_event(source=input_type, index=helper.get_output_index(stanza_name), sourcetype=helper.get_sourcetype(stanza_name), data=data)
        ew.write_event(event)
    '''

```
