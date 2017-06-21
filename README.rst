Introduction
============

This project allows the Python API from Interactive Brokers (IBAPI)
to be used asynchronously with the
`asyncio <https://docs.python.org/3.5/library/asyncio.html>`
standard library or with
`PyQt5 <https://pypi.python.org/pypi/PyQt5>`.

This offers a simpler, safer and more performant approach to concurrency than
multithreading.


Installation
============

::
    pip3 -U tws_async

Note that on some systems the ``pip3`` command is just ``pip``.

`Python <http://www.python.org>` version 3.5 or higher is required as well as the
`Interactive Brokers Python API <http://interactivebrokers.github.io>`.


Usage
=====

This package offers two clients that can be used as a drop-in replacement for
the standard EClient as provided by IBAPI:
- TWSClient, for use with the asyncio event loop;
- TWSClientQt, for use with the PyQt5 event loop.

These clients also inherit from ibapi.wrapper.EWrapper and can be used exactly
as one would use the standard IBAPI version. The asynchronous clients use
their own event-driven networking code that replaces the networking code
of the standard EClient, and they also replace the infinite loop of
EClient.run() with an event loop.

To simplify working with contracts, this package provides
Contract, Stock, Option, Future, Forex, Index, CFD and Commodity
classes that can be used anywhere where a ibapi.contract.Contract is expected.
Examples of some simple cases are
Stock('AMD'), Forex('EURUSD'), CFD('IBUS30') or Future('ES', '201612', 'GLOBEX')
To specify more complex contracts, any property can be given as a keyword.

To learn more, consult the
`official IBAPI documentation <https://interactivebrokers.github.io/tws-api/#gsc.tab=0>`
or have a look at these sample usecases:

Historical data downloader
--------------------------
The `HistRequester <tws_async/histrequester.py>` downloads historical data and
saves it to CSV files;
`histrequester demo <samples/histrequester_demo.py>` illustrates how to use it.

Realtime streaming ticks
------------------------
The `tick streamer <samples/tickstreamer_demo.py` subscribes
to realtime tick data.

Jupyter Notebook
----------------
To use the Interactive Brokers API fully interactively in a Jupyter notebook,
have a look at the `example notebook <samples/tws.ipynb>`.

Jupyter can be started with the command::

    jupyter notebook

This notebook uses the Qt version of the client, where the
Qt event loop is started with the ``%gui qt5`` directive at the very top.
It is not necessary to call the run() method of the client.

Notes on using asycio in a notebook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Currently there does not seem to be a single-threaded way to directly run
the asyncio event loop in Jupyter. What can be done is to use the
Qt event loop (which does have good integration with the Jupyter kernel)
with the `quamash <https://github.com/harvimt/quamash>`adaptor.
With quamash the Qt event loop is used to drive the asyncio event loop.
It can be done by placing this code at the top of the notebook:
::code-block python
    %gui qt5
    import asyncio
    import quamash
    loop = quamash.QEventLoop()
    asyncio.set_event_loop(loop)
With this it is possible to run the asyncio version of the client as well.

One thing that does not work in the combination of quamash and Jupyter is the
loop.run_until_finished method. It can be patched like this:
::code-block python
    def run_until_complete(self, future):
        future = asyncio.ensure_future(future)
        qApp = qt.QApplication.instance()
        while(not future.done()):
            qApp.processEvents()
            qt.QThread.usleep(1000)
        qApp.processEvents()
        return future.result()
    
    quamash.QEventLoop.run_until_complete = run_until_complete


