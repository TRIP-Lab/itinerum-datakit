Installation
============

The **itinerum-tripkit** can be installed as a library using pip can be included in your own project by cloning: http://github.com/TRIP-Lab/itinerum-tripkit and
copying the ``tripkit`` directory as a submodule.


Virtual Environments
--------------------
It is recommended to use venv_ to keep the tripkit dependency versions isolated from system packages.

Linux & MacOS
+++++++++++++
.. code-block:: bash

    $ python3 -m venv tripkit-venv
    $ source ./tripkit-ven/bin/activate

Windows
+++++++

.. code-block:: powershell

    PS C:\Code\itinerum-tripkit> python -m venv tripkit-venv
    PS C:\Code\itinerum-tripkit> .\tripkit-venv\Scripts\Activate.ps1

**PowerShell Note:**
With PowerShell, ``Set-ExecutionPolicy Unrestricted -Force`` may be required to allow the ``Activate.ps1`` 
script to run. The PowerShell prompt must then be restarted for these permissions to take effect.

Dependencies
------------
Windows
+++++++
Windows does not provide a build environment by default with libraries relied upon by the GDAL package. Instead, the 
`Visual C++ Redistributable for Visual Studio 2015`_ (13.4MB) can be installed to provide the necessary system libraries.

If an existing GDAL installation is available to Python, it may be possible to skip this step.

Compiled Python Packages
~~~~~~~~~~~~~~~~~~~~~~~~
On Windows without a C/C++ build environment, some packages requiring will fail to install using `pip`. Instead, compiled wheel versions can be
downloaded from various mirrors and installed with pip directly from file.

.. code-block:: powershell

    (tripkit-venv) PS C:\Code\itinerum-tripkit> pip install .\Fiona-1.8.6-cp37-cp37m-win_amd64.whl

Compiled packages to install:

* GDAL: https://www.lfd.uci.edu/~gohlke/pythonlibs/#gdal
* Fiona: https://www.lfd.uci.edu/~gohlke/pythonlibs/#fiona


GDAL (Alternative)
^^^^^^^^^^^^^^^^^^
If the above is not successful, another method is using the gisinternals.com pre-compiled binaries. For your system version (likely *MSVC 2017 / x64*), click "Downloads". From
the downloads page, the core GDAL library is all that is needed (*gdal-204-1911-64-core.msi*).

Install this file and set two Windows environment variables:

- Append to PATH: ``C:\Program Files\GDAL``
- Create GDAL_DATA: ``C:\Program Files\GDAL\gdal-data``

After setting these variables, close and re-open the command prompt (re-activate the virtual environment if using) and the Python dependencies can be installed.

First the GDAL library must be installed for geospatial operations and outputs. 



Optional Components
~~~~~~~~~~~~~~~~~~~
Scikit-learn
^^^^^^^^^^^^

Scikit-learn can optionally be installed for optimized clustering such as by swapping out the included K-Means implementation:

* Scikit-learn: https://www.lfd.uci.edu/~gohlke/pythonlibs/#scikit-learn


OSRM
^^^^

The **itinerum-tripkit** provides interfaces for submitting map matching queries to an OSRM API and writing results to file.

The instructions that follow use the `Multi-Level Djikstra processing pipelines`_ recommended by Project OSRM.

Installing the OSRM API with Docker containers
``````````````````````````````````````````````

1. Download an OSM extract for your region (ex. Québec)

.. code-block:: bash

   $ mkdir osrm && cd osrm
   $ wget http://download.geofabrik.de/north-america/canada/quebec-latest.osm.pbf


2. Process the OSM data using the default network profiles included with OSRM:

.. code-block:: bash

   # car
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-extract -p /opt/car.lua /data/quebec-latest.osm.pbf
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-partition /data/quebec-latest
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-customize /data/quebec-latest
   $ mkdir car
   $ mv quebec-latest.orsm* car
   
   # bike
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-extract -p /opt/bicycle.lua /data/quebec-latest.osm.pbf
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-partition /data/quebec-latest
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-customize /data/quebec-latest
   $ mkdir bicycle
   $ mv quebec-latest.orsm* bicycle
   
   # walking
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-extract -p /opt/foot.lua /data/quebec-latest.osm.pbf
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-partition /data/quebec-latest
   $ docker run -t -v $(pwd):/data osrm/osrm-backend osrm-customize /data/quebec-latest
   $ mkdir foot
   $ mv quebec-latest.orsm* foot

3. Run the Docker OSRM API containers on ports ``5000-5002`` to reverse proxy for public access

.. code-block:: bash

   $ docker run -d --restart always -p 5000:5000 -v $(pwd)/car:/data osrm/osrm-backend osrm-routed --algorithm MLD --max-matching-size=5000 /data/quebec-latest.osrm
   
   $ docker run -d --restart always -p 5001:5000 -v $(pwd)/bicycle:/data osrm/osrm-backend osrm-routed --algorithm MLD --max-matching-size=5000 /data/quebec-latest.osrm
   
   $ docker run -d --restart always -p 5002:5000 -v $(pwd)/foot:/data osrm/osrm-backend osrm-routed --algorithm MLD --max-matching-size=5000 /data/quebec-latest.osrm


.. _venv: https://docs.python.org/3/library/venv.html
.. _Multi-Level Djikstra processing pipelines: https://github.com/Project-OSRM/osrm-backend/wiki/Running-OSRM
.. _Visual C++ Redistributable for Visual Studio 2015: https://www.microsoft.com/en-ca/download/details.aspx?id=48145