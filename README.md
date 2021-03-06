Python-Zimbra
=============

| cPython 2.7       | cPython 3.2       | cPython 3.3.6       | cPython 3.4.3       | PyPy 2.6.1              | PyPy3 2.4.0               |
|-------------------|-------------------|---------------------|---------------------|-------------------------|---------------------------|
| [![ibs27]] [bs27] | [![ibs32]] [bs32] | [![ibs336]] [bs336] | [![ibs343]] [bs343] | [![ibspy261]] [bspy261] | [![ibspy3240]] [bspy3240] |
| [![icv27]] [cv27] | [![icv32]] [cv32] | [![icv336]] [cv336] | [![icv343]] [cv343] | [![icvpy261]] [cvpy261] | [![icvpy3240]] [cvpy3240] | 

Python classes to access Zimbra SOAP backend with a few utilities. Handles
creating and sending Zimbra SOAP queries to the backend and adds a few
utilities such as a preauth generator.

Compatible with Python 2.7 and 3.x (cPython and PyPy)

This framework is not intended to supply user level access to Zimbra
functions. Zimbra tends to be too dynamic and complex for this to work. This
is actually a framework to reduce the implementation work if you want to
speak to the Zimbra SOAP API.

Please refer to the [official SOAP documentation by Zimbra](http://wiki.zimbra.com/wiki/SOAP_API_Reference_Material_Beginning_with_ZCS_8.0)
on how to use the SOAP backend.

For details refer to the [API-Documentation](http://zimbra-community.github.io/python-zimbra/docs/)

Installation
------------

To install, either use things like easy_install or pip with the package
"python-zimbra". (You may have to add --pre to download a prerelease) or
download the package and put the "pythonzimbra" directory into your python
path.

Tutorial
--------

If you'd like to get information about a folder of a specific user, 
this is how you do it:

First, import the needed libraries (You will learn about them later):

    from pythonzimbra.tools import auth
    from pythonzimbra.communication import Communication

Let's assume you have a variable called "url" which holds the URL to your
Zimbra server. For our example this has to be the URL for the user backend, 
that is typically `https://your-zimbra-server/service/soap`.

Now, build up the communication object:

    comm = Communication(url)

This will be used to send the request later on. But first,
we have to authenticate using [Zimbra preauth](https://wiki.zimbra.com/wiki/Preauth).
We can do this by using the auth-helper:

    token = auth.authenticate(
        url,
        'myuser@mydomain.com',
        'fkiwfki2ri32fiqepnfpwenufpsecretpreauthkey'
    )

This should return the authentication token to be used in Zimbra requests. If
 it returns None, the authentication is somehow failed (maybe wrong username,
  URL or preauthentication-key)

Now, we create our request

    info_request = comm.gen_request(token=token)

and add our (very simple) GetFolder request,
which is in the urn:zimbraMail-namespace:

    info_request.add_request(
        "GetFolderRequest",
        {
            "folder": {
                "path": "/inbox"
            }
        },
        "urn:zimbraMail"
    )

Finally, we sent the request:

    info_response = comm.send_request(info_request)

The info_response-variable now holds the response information of the object.

Now, if the response was successful and has now Fault-object:

    if not info_response.is_fault():

Print the message count of that folder:

    print info_response.get_response()['GetFolderResponse']['folder']['n']

Batch requests
--------------

Working with batch requests is also possible. To do that, set the
parameter "set_batch":

    batch_request = comm.gen_request(set_batch=True)

And can afterwards add multiple requests using add_request to it. You'll get 
the request id of the specific request as a return value. Use that id to 
retrieve the response later using get_response(id).

Working with faults
-------------------

If you get a response, that resulted in a fault, you can check that with:

    response.is_fault()
    
To get further information about the response, use the two methods

    response.get_fault_code()
    response.get_fault_message()
    
The fault_code is Zimbra's own fault message code (like mail.NO_SUCH_FOLDER).
 The message is a more elaborate message like (no such folder path: /...).

Authentication against the administration console
-------------------------------------------------

Zimbra currently doesn't support the preauth-method for authentications against
the admin-console (URL `https://your-zimbra-server:7071/service/admin/soap`).

python-zimbra's auth tool can be used to authenticate to this url by specifying
the password instead of the preauth-key and setting the parameter admin_auth to
True. (see API docs for specifics)

Used dictionary
---------------

All requests and responses are built up using a certain dictionary format.
This is heavily influenced by the Zimbra json format being:

    {
        "RequestName": {
            "_content": "Content of the node"
            "attribute": "value",
            "subnode": {
                "_content": "Content of the subnode"
            }
        }
    }

in XML this would look like this:

    <RequestName attribute="value">
        <subnode>
            Content of the subnode
        </subnode>

        Content of the node
    </RequestName>

All requests should conform to this dictionary format and the responses are
also returned in this format. Subnodes can also contain lists of
dictionaries, which will create multiple subnodes with the same tag.

Warning
-------

There are missing sanity checks on purpose. This is because the Zimbra API is
 due to heavy modifications and to keep up with the current API catalogue is
 quiet problematic.

So please be aware of this and use the library with caution. Don't expose the
 library to the public without doing sanity checks on your own.

Testing
-------

Python-Zimbra includes a testsuite with unittests, that test the supported
features.

To enable testing in your environment, copy the config.ini.dist to config.ini
 in the tests module and configure it to match your environment.

For some tests to work you need a Zimbra server with an admin and a
user account. You have to specifically enable these tests. Here's an overview
 of what is done using your zimbra server inside these tests:

* test_admin.py
  * Authenticate as admin
  * Add a test account
  * Try logging in using that test account
  * Delete the test account
* test_auth.py
  * Authenticate as user
  * Authenticate as user with wrong preauth key
  * Authenticate as user with password
  * Authenticate as user with wrong password
* test_autoresponse.py
  * Authenticate as user
  * Send NoOpRequest
* test_fault.py
  * Authenticate as user
  * Query a non-existing folder using GetFolderRequest
  * Query a non-existing folder using GetFolderRequest inside a BatchRequest
* test_genrequest.py
  * Authenticate as user
  * Send a NoOpRequest
  * Send a NoOpRequest inside a BatchRequest
  * Send a GetInfoRequest
  * Send a NoOpRequest and a GetInfoRequest inside a BatchRequest

To run the test, enter the tests subdirectory and run

    python -m unittest discover -s ..

We thankfully use [Travis](travis-ci.org) for continuous integration and
[Coveralls](https://coveralls.io) for code coverage.

The Zimbra server used in CI-testing is kindly hosted by [efm](http://www.efm.de/).

[ibs27]: https://img.shields.io/teamcity/http/ci.blueocean-net.de/s/ZimbraCommunity_PythonZimbra_TestPy27.png
[bs27]: http://ci.blueocean-net.de/viewType.html?buildTypeId=ZimbraCommunity_PythonZimbra_TestPy27
[icv27]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPy27/.lastFinished/tests/coverage.png/coverage.png
[cv27]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPy27/.lastFinished/tests/htmlcov/index.html 

[ibs32]: https://img.shields.io/teamcity/http/ci.blueocean-net.de/s/ZimbraCommunity_PythonZimbra_TestPy32.png
[bs32]: http://ci.blueocean-net.de/viewType.html?buildTypeId=ZimbraCommunity_PythonZimbra_TestPy32
[icv32]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPy32/.lastFinished/tests/coverage.png/coverage.png
[cv32]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPy32/.lastFinished/tests/htmlcov/index.html

[ibs336]: https://img.shields.io/teamcity/http/ci.blueocean-net.de/s/ZimbraCommunity_PythonZimbra_TestPy336.png
[bs336]: http://ci.blueocean-net.de/viewType.html?buildTypeId=ZimbraCommunity_PythonZimbra_TestPy336
[icv336]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPy336/.lastFinished/tests/coverage.png/coverage.png
[cv336]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPy336/.lastFinished/tests/htmlcov/index.html
 
[ibs343]: https://img.shields.io/teamcity/http/ci.blueocean-net.de/s/ZimbraCommunity_PythonZimbra_TestPy343.png
[bs343]: http://ci.blueocean-net.de/viewType.html?buildTypeId=ZimbraCommunity_PythonZimbra_TestPy343
[icv343]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPy343/.lastFinished/tests/coverage.png/coverage.png
[cv343]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPy343/.lastFinished/tests/htmlcov/index.html
 
[ibspy261]: https://img.shields.io/teamcity/http/ci.blueocean-net.de/s/ZimbraCommunity_PythonZimbra_TestPypy261.png
[bspy261]: http://ci.blueocean-net.de/viewType.html?buildTypeId=ZimbraCommunity_PythonZimbra_TestPypy261
[icvpy261]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPypy261/.lastFinished/tests/coverage.png/coverage.png
[cvpy261]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPypy261/.lastFinished/tests/htmlcov/index.html 

[ibspy3240]: https://img.shields.io/teamcity/http/ci.blueocean-net.de/s/ZimbraCommunity_PythonZimbra_TestPypy3240.png
[bspy3240]: http://ci.blueocean-net.de/viewType.html?buildTypeId=ZimbraCommunity_PythonZimbra_TestPypy3240
[icvpy3240]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPypy3240/.lastFinished/tests/coverage.png/coverage.png
[cvpy3240]: http://ci.blueocean-net.de/repository/download/ZimbraCommunity_PythonZimbra_TestPypy3240/.lastFinished/tests/htmlcov/index.html