Pilbox
======

Pilbox is an image resizing application server built on Python's [Tornado web framework](http://www.tornadoweb.org/en/stable/) using the [Python Imaging Library (PIL)](http://www.pythonware.com/products/pil/). It is not intended to be the primary source of images, but instead acts as a proxy which requests images and resizes them as desired.

[![Build Status](https://travis-ci.org/agschwender/pilbox.png)](https://travis-ci.org/agschwender/pilbox)

Setup
=====

Dependencies
------------

  * [Python 2.7](http://www.python.org/download/)
  * [PIL 1.1.7](http://www.pythonware.com/products/pil/)
  * [tornado 3.1](https://pypi.python.org/pypi/tornado/3.1)
  * Image Libraries: libjpeg-dev, libfreetype6, libfreetype6-dev, zlib1g-dev

Vagrant
-------

Packaged with Pilbox is a [Vagrant](http://www.vagrantup.com/) configuration file which installs all necessary dependencies on a virtual box. See the [Vagrant documentation for installation instructions](http://docs.vagrantup.com/v2/installation/). Once installed, the following will start a virtual machine.

    $ vagrant up

To access the virtual machine itself, simply...

    $ vagrant ssh

Running
=======

Manual
------

To run the application, issue the following command

    $ python pilbox/app.py

By default, this will run the application on port 8888 and can be accessed by visiting:

    http://localhost:8888/

To see a list of all available options, run

    $ python pilbox/app.py --help

Vagrant
-------

When running via Vagrant, the application is automatically started via [Supervisor](http://supervisord.org/). The Vagrant setup runs the application behind [Nginx](http://nginx.org/) which caches the output via [Varnish](https://www.varnish-cache.org/).

If accessing the application via Vagrant, you will need to determine the virtual machine's IP address.

    $ vagrant ssh
    $ /sbin/ifconfig -a

Once determined, the application can be accessed via port 80, e.g.

    http://192.168.1.1/

Calling
=======

To use the image resizing service, include the application url as you would any other image, e.g.

    <img src="http://localhost:8888/?url=http%3A%2F%2Fi.imgur.com%2FzZ8XmBA.jpg&w=300&h=300&mode=crop" width="300" height="300" />

This will request the image served at the supplied url and resize it to 300x300 using the crop mode. The following is the list of parameters that can be supplied to the service

  * _url_: The url of the image to be resized
  * _w_: The desired width of the image
  * _h_: The desired height of the image
  * _mode_: The resizing method: clip, crop (default) and scale
    * _clip_: Resize to fit within the desired region, keeping aspect ratio
    * _crop_: Resize so one dimension fits within region, center, cut remaining
    * _scale_: Resize to fit within the desired region, ignoring aspect ratio
  * _client_: The client name
  * _sig_: The signature

The `url`, `w` and `h` parameters are required. `mode` is optional and defaults to `crop`. `client` is required only if the `client_name` is defined within the configuration file. Likewise, `sig` is required only if the `client_key` is defined within the configuration file. See the [signing section](#signing) for details on how to generate the signature.

Testing
=======

To run all tests, issue the following command

    $ python -m pilbox.test.runtests

To run individual tests, simply indicate the test to be run, e.g.

    $ python -m pilbox.test.runtests pilbox.test.signature_test

Signing
=======

In order to secure requests so that unknown third parties cannot easily use the resize service, the application can require that requests provide a signature. To enable this feature, set the `client_key` option. The signature is a hexadecimal digest generated from the client key and the query string using the HMAC-SHA1 message authentication code (MAC) algorithm. The below python code provides an example implementation.

    import hashlib
    import hmac

    def derive_signature(key, qs):
        m = hmac.new(key, None, hashlib.sha1)
        m.update(qs)
        return m.hexdigest()

The signature is passed to the application by appending the `sig` paramater to the query string; e.g. `x=1&y=2&z=3&sig=c9516346abf62876b6345817dba2f9a0c797ef26`. Note, the application does not include the leading question mark when verifying the supplied signature. To verify your signature implementation, see the `pilbox.signature` command described in the [tools section](#tools).

Tools
=====

To verify that your client application is generating correct signatures, use the signature command.

    $ python -m pilbox.signature --key=abcdef "x=1&y=2&z=3"
    Query String: x=1&y=2&z=3
    Signature: c9516346abf62876b6345817dba2f9a0c797ef26
    Signed Query String: x=1&y=2&z=3&sig=c9516346abf62876b6345817dba2f9a0c797ef26

The application allows the use of the resize functionality via the command line.

    $ python -m pilbox.image --width=300 --height=300 http://i.imgur.com/zZ8XmBA.jpg > /tmp/foo.jpg

TODO
====

  * Add controller tests
  * Build-in automatic deploy to ec2 instance
  * Fill resize with background
  * Crop resize positioning
  * Add backends (S3, file system, etc...) if necessary