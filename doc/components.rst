.. _components:
.. include:: globals.rst

#######################
 Network and components
#######################
|doc_version|

.. note::	Hyperlinks in the text that follows are common terms of
			in PyPSA vocabulary and link to short definitions in the 
			:ref:`Glossary<glossary>`.  If you are new to PyPSA, you might find it
			useful to first review :ref:`Network and key PyPSA terminology<terminology>`

Network
=======

The table below lists the primary data attributes for the :term:`Network` class object.  
For a full list of its methods, e.g. ``network.lopf()`` and ``network.pf()``,
see the :ref:`Network API reference<api>`.  Additional attributes, e.g. ``results`` and ``objective``,
are pre-cursors or defined by some of these methods.

Certain of the attributes reference a syntax borrowing on Python f-string notation, i.e. string
concatenation with variables enclosed in {}s.  For instance, ``{list_name}_t`` indicates a set of attributes is formed
fronm each :term:`list_name` variable concatenated with "_t", e.g. ``network.generators_t``, ``network.loads_t``,
``network.lines_t``, and so on are all available attributes for all defined ``list_name`` variables.

The ``list_name`` variable for each A :term:`component` is used to reference an equivalently
named attributes definition csv file located in the ``component_attrs`` sub-diretory.
This file specifies the specifics of all attributes defined for each component.
Each of the Component-specific sections that follow on this page are essentially a short
summary and explicit embed of these attribute definition csv tables.

References to :term:`list_name` reference each of the components defined in ``components.csv``
located in the package main folder.  These ``list_names`` in turn are used to
reference associated filenames in the ``component_attrs`` sub-directory.  

.. csv-table::
   :header-rows: 1
   :width: 156
   :widths: 20, 14, 14, 14, 80, 14
   :file: ../doc/tables/network_doc.csv


.. _component-attrs:

Components Schema: ``network.components`` dictionary 
================================================

The schema data for each Component class is stored in a nested dictionary assigned to the
network object's ``.components`` attribute, which has keys corresponding to the
``Component`` string accessors, e.g. ``network.components.Bus`` or ``network.components["Bus"]``
will access the meta information for the Bus class of devices.  The inner dictionaries are formed
from a meld of the values defined in ``components.csv`` file and the attribute-specific information
for the Component defined in its csv in ``component_attrs`` sub-directory.  So, for example:
``network.components["Generator"]["type"]`` returns the string "controllable_one_port".

The attribute-specific data is held in a ``pandas.DataFrame`` accessed via the "attrs" key, so that
``network.components["Generator"]["attrs"]`` returns a ``DataFrame`` listing all attributes, and 
meta information for those attributes, for all devices defined within the Component class "Generator".
This DataFrame uses data defined within the csv file (corresponding to the component's
``list_name``) in ``component_attrs`` as a starting point, and then is amended and expanded
by the PyPSA code yielding columns and their meaning as described in the table below.

.. attention::	This table is in very 'raw form' based on my own personal documentaiton.  Work
				in progress.  Once sorted it out, needs a re-write into more succinct form that simply
				summarizes the outcome

``DataFrame`` structure for ``network.components[Component]["attrs"]``:

.. csv-table::
   :header-rows: 1
   :file: ../doc/tables/component_attrs.csv

.. attention::  End point on this .rst file of review and intervention for provisional |doc_version|
				
Components
==========

PyPSA represents the power system using the following :term:`components`:

.. csv-table::
   :header-rows: 1
   :file: ../pypsa/components.csv

This information is also available within a dictionary within each network
object as ``network.components``.  (See Section **"network.components dictionary"** above.)

For functions such as :doc:`power_flow` and :doc:`optimal_power_flow` the inputs used
and outputs given are listed in their documentation.


Sub-Network
===========

Sub-networks are determined by PyPSA and should not be entered by the
user.

Sub-networks are subsets of buses and passive branches (i.e. lines and
transformers) that are connected.

They have a uniform energy``carrier`` inherited from the buses, such as
"DC", "AC", "heat" or "gas". In the case of "AC" sub-networks, these
correspond to synchronous areas. Only "AC" and "DC" sub-networks can
contain passive branches; all other sub-networks must contain a single
isolated bus.

The power flow in sub-networks is determined by the passive flow
through passive branches due to the impedances of the passive branches.

Sub-Network are determined by calling
``network.determine_network_topology()``.


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/sub_networks.csv


Bus
===

The bus is the fundamental node of the network, to which components
like loads, generators and transmission lines attach. It enforces
energy conservation for all elements feeding in and out of it
(i.e. like Kirchhoff's Current Law).


.. image:: img/buses.png




.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/buses.csv



Carrier
=======

The carrier describes energy carriers and defaults to ``AC`` for
alternating current electricity networks. ``DC`` can be set for direct
current electricity networks. It can also take arbitrary values for
arbitrary energy carriers, e.g. ``wind``, ``heat``, ``hydrogen`` or
``natural gas``.

Attributes relevant for global constraints can also be stored in this
table, the canonical example being CO2 emissions of the carrier
relevant for limits on CO2 emissions.


.. note:: In versions of PyPSA < 0.6.0, this was called Source.


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/carriers.csv



.. _global-constraints:

Global Constraints
==================

Global constraints are added to OPF problems and apply to many
components at once. Currently only constraints related to primary
energy (i.e. before conversion with losses by generators) are
supported, the canonical example being CO2 emissions for an
optimisation period. Other primary-energy-related gas emissions also
fall into this framework.

Other types of global constraints will be added in future, e.g. "final
energy" (for limits on the share of renewable or nuclear electricity
after conversion), "generation capacity" (for limits on total capacity
expansion of given carriers) and "transmission capacity" (for limits
on the total expansion of lines and links).

.. note:: Global constraints were added in PyPSA 0.10.0 and replace the ad hoc ``network.co2_limit`` attribute.


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/global_constraints.csv


Generator
=========

Generators attach to a single bus and can feed in power. It converts
energy from its ``carrier`` to the carrier-type of the bus to which it
is attached.

In the LOPF the limits which a generator can output are set by
``p_nom*p_max_pu`` and ``p_nom*p_min_pu``, i.e. by limits defined per
unit of the nominal power ``p_nom``.


Generators can either have static or time-varying ``p_max_pu`` and
``p_min_pu``.

Generators with static limits are like controllable conventional
generators which can dispatch anywhere between ``p_nom*p_min_pu`` and
``p_nom*p_max_pu`` at all times. The static factor ``p_max_pu``,
stored at ``network.generator.loc[gen_name,"p_max_pu"]`` essentially
acts like a de-rating factor. In the following example ``p_max_pu =
0.9`` and ``p_min_pu = 0``. Since ``p_nom`` is 12000 MW, the maximum
dispatchable active power is 0.9*12000 MW = 10800 MW.

.. image:: img/nuclear-dispatch.png


Generators with time-varying limits are like variable
weather-dependent renewable generators. The time series ``p_max_pu``,
stored as a series in ``network.generators_t.p_max_pu[gen_name]``,
dictates the active power availability for each snapshot per unit of
the nominal power ``p_nom`` and another time series ``p_min_pu`` which
dictates the minimum dispatch. These time series can take values
between 0 and 1, e.g. ``network.generators_t.p_max_pu[gen_name]``
could be

.. image:: img/p_max_pu.png

This time series is then multiplied by ``p_nom`` to get the available
power dispatch, which is the maximum that may be dispatched. The
actual dispatch ``p``, stored in ``network.generators_t.p[gen_name]``,
may be below this value, e.g.

.. image:: img/scigrid-curtailment.png


For the implementation of unit commitment, see :ref:`unit-commitment`.

For generators, if :math:`p>0` the generator is supplying active power
to the bus and if :math:`q>0` it is supplying reactive power
(i.e. behaving like a capacitor).


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/generators.csv



Storage Unit
============

Storage units attach to a single bus and are used for inter-temporal
power shifting. Each storage unit has a time-varying state of charge
and various efficiencies. The nominal energy is given as a fixed ratio
``max_hours`` of the nominal power. If you want to optimise the
storage energy capacity independently from the storage power capacity,
you should use a fundamental ``Store`` component (see below) attached
with two ``Link`` components, one for charging and one for
discharging. See also the `example that replaces generators and
storage units with fundamental links and stores
<https://pypsa.org/examples/replace-generator-storage-units-with-store.html>`_.


For storage units, if :math:`p>0` the storage unit is supplying active
power to the bus and if :math:`q>0` it is supplying reactive power
(i.e. behaving like a capacitor).



.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/storage_units.csv


Store
=====

The ``Store`` connects to a single bus. It is a more fundamental
component for storing energy only (it cannot convert between energy
carriers). It inherits its energy carrier from the bus to which it is
attached.

The Store, Bus and Link are fundamental components with which one can
build more complicated components (Generators, Storage Units, CHPs,
etc.).

The Store has controls and optimisation on the size of its energy
capacity, but not it's power output; to control the power output, you
must put a link in front of it, see the `example that replaces
generators and storage units with fundamental links and stores
<https://pypsa.org/examples/replace-generator-storage-units-with-store.html>`_.



.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/stores.csv


Load
====

The load attaches to a single bus and consumes power as a PQ load.

For loads, if :math:`p>0` the load is consuming active power from the
bus and if :math:`q>0` it is consuming reactive power (i.e. behaving
like an inductor).


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/loads.csv


Shunt Impedance
===============

Shunt impedances attach to a single bus and have a voltage-dependent
admittance.

For shunt impedances the power consumption is given by :math:`s_i =
|V_i|^2 y_i^*` so that :math:`p_i + j q_i = |V_i|^2 (g_i
-jb_i)`. However the p and q below are defined directly proportional
to g and b :math:`p = |V|^2g` and :math:`q = |V|^2b`, thus if
:math:`p>0` the shunt impedance is consuming active power from the bus
and if :math:`q>0` it is supplying reactive power (i.e. behaving like
an capacitor).


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/shunt_impedances.csv


Line
====

Lines represent transmission and distribution lines. They connect a
``bus0`` to a ``bus1``. They can connect either AC buses or DC
buses. Power flow through lines is not directly controllable, but is
determined passively by their impedances and the nodal power
imbalances. To see how the impedances are used in the power flow, see
:ref:`line-model`.


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/lines.csv


.. _line-types:

Line Types
==========

Standard line types with per length values for impedances.

If for a line the attribute "type" is non-empty, then these values are
multiplied with the line length to get the line's electrical
parameters.

The line type parameters in the following table and the implementation
in PyPSA are based on `pandapower's standard types
<https://pandapower.readthedocs.io/en/latest/std_types/basic.html>`__,
whose parameterisation is in turn loosely based on `DIgSILENT
PowerFactory
<http://www.digsilent.de/index.php/products-powerfactory.html>`_.


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/line_types.csv


If you do not import your own line types, then PyPSA will provide
standard types using the following table. We thank the pandapower team for allowing us to include this data.
We take no responsibility for the accuracy of the values.

.. csv-table::
   :header-rows: 1
   :file: ../pypsa/standard_types/line_types.csv


Transformer
===========

Transformers represent 2-winding transformers that convert AC power
from one voltage level to another. They connect a ``bus0`` (typically at higher voltage) to a
``bus1`` (typically at lower voltage). Power flow through transformers is not
directly controllable, but is determined passively by their impedances
and the nodal power imbalances. To see how the impedances are used in
the power flow, see :ref:`transformer-model`.


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/transformers.csv


.. _transformer-types:

Transformer Types
=================

Standard 2-winding transformer types.

If for a transformer the attribute "type" is non-empty, then these
values are used for the transformer's electrical parameters.


The transformer type parameters in the following table and the
implementation in PyPSA are based on `pandapower's standard
types
<http://www.uni-kassel.de/eecs/fileadmin/datas/fb16/Fachgebiete/energiemanagement/Software/pandapower-doc/std_types/basic.html>`_,
whose parameterisation is in turn loosely based on `DIgSILENT
PowerFactory
<http://www.digsilent.de/index.php/products-powerfactory.html>`_.

.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/transformer_types.csv



If you do not import your own transformer types, then PyPSA will
provide standard types using the following table. This table was
initially based on `pandapower's standard types
<http://www.uni-kassel.de/eecs/fileadmin/datas/fb16/Fachgebiete/energiemanagement/Software/pandapower-doc/std_types/basic.html>`_
and we thank the pandapower team for allowing us to include this data.
We take no responsibility for the accuracy of the values.


.. csv-table::
   :header-rows: 1
   :file: ../pypsa/standard_types/transformer_types.csv


.. _controllable-link:

Link
====

The ``Link`` is a component introduced in PyPSA 0.5.0 for controllable
directed flows between two buses ``bus0`` and ``bus1`` with arbitrary
energy carriers. It can have an efficiency loss and a marginal cost;
for this reason its default settings allow only for power flow in one
direction, from ``bus0`` to ``bus1`` (i.e. ``p_min_pu = 0``). To build
a bidirectional lossless link, set ``efficiency = 1``, ``marginal_cost
= 0`` and ``p_min_pu = -1``.

The ``Link`` component can be used for any element with a controllable
power flow: a bidirectional point-to-point HVDC link, a unidirectional
lossy HVDC link, a converter between an AC and a DC network, a heat
pump or resistive heater from an AC/DC bus to a heat bus, etc.

.. note:: ``Link`` has replaced the ``Converter`` component for linking AC with DC buses and the ``TransportLink`` component for providing controllable flows between AC buses. If you want to replace ``Converter`` and ``TransportLink`` components in your old code, use the ``Link`` with ``efficiency = 1``, ``marginal_cost = 0``, ``p_min_pu = -1``, ``p_max_pu = 1`` and ``p_nom* = s_nom*``.

.. csv-table::
   :header-rows: 1
   :file: ../pypsa/component_attrs/links.csv


.. _components-links-multiple-outputs:

Link with multiple outputs or inputs
------------------------------------

Links can also be defined with multiple outputs in fixed ratio to the
power in the single input by defining new columns ``bus2``, ``bus3``,
etc. (``bus`` followed by an integer) in ``network.links`` along with
associated columns for the efficiencies ``efficiency2``,
``efficiency3``, etc. The different outputs are then equal to
the input multiplied by the corresponding efficiency; see :ref:`opf-links` for how
these are used in the LOPF and the `example of a CHP with a fixed
power-heat ratio
<https://www.pypsa.org/examples/chp-fixed-heat-power-ratio.html>`_.

To define the new columns ``bus2``, ``efficiency2``, ``bus3``,
``efficiency3``, etc. in ``network.links`` you need to override the
standard component attributes by passing ``pypsa.Network()`` an
``override_component_attrs`` argument. See the section
:ref:`custom_components` and the `example of a CHP with a fixed
power-heat ratio
<https://www.pypsa.org/examples/chp-fixed-heat-power-ratio.html>`_.


If the column ``bus2`` exists, values in the column are not compulsory
for all links; if the link has no 2nd output, simply leave it empty
``network.links.at["my_link","bus2"] = ""``.

For links with multiple inputs in fixed ratio to one of the inputs,
you can define the other inputs as outputs with a negative efficiency
so that they withdraw energy or material from the bus if there is a positive
flow in the link.

As an example, suppose a link representing a methanation process takes
as inputs one unit of hydrogen and 0.5 units of carbon dioxide, and
gives as outputs 0.8 units of methane and 0.2 units of heat. Then
``bus0`` connects to hydrogen, ``bus1`` connects to carbon dioxide
with ``efficiency=-0.5`` (since 0.5 units of carbon dioxide is taken
for each unit of hydrogen), ``bus2`` connects to methane with
``efficiency2=0.8`` and ``bus3`` to heat with ``efficiency3=0.2``.

The Jupyter notebook `Biomass, synthetic fuels and carbon management <https://github.com/PyPSA/PyPSA/blob/master/examples/notebooks/biomass-synthetic-fuels-carbon-management.ipynb>`_ provides many examples of modelling processes with multiple inputs and outputs using links.

Groups of Components
====================

In the code components are grouped according to their properties in
sets such as ``network.one_port_components`` and
``network.branch_components``.

One-ports share the property that they all connect to a single bus,
i.e. generators, loads, storage units, etc.. They share the attributes
``bus``, ``p_set``, ``q_set``, ``p``, ``q``.

Branches connect two buses. A copy of their attributes can be accessed
as a group by the function ``network.branches()``. They share the
attributes ``bus0``, ``bus1``.

Passive branches are branches whose power flow is not directly
controllable, but is determined passively by their impedances and the
nodal power imbalances, i.e. lines and transformers.

Controllable branches are branches whose power flow can be controlled
by e.g. the LOPF optimisation, i.e. links.


.. _custom_components:

Custom Components
=================

If you want to define your own components and override the standard
functionality of PyPSA, you can easily override the standard
components by passing pypsa.Network() the arguments
``override_components`` and ``override_component_attrs``.

For this network, these will replace the standard definitions in
``pypsa.components.components`` and
``pypsa.components.component_attrs``, which correspond to the
repository CSV files ``pypsa/components.csv`` and
``pypsa/component_attrs/*.csv``.

``components`` is a pandas.DataFrame with the component ``name``,
``list_name`` and ``description``. ``component_attrs`` is a
pypsa.descriptors.Dict of pandas.DataFrame with the attribute
properties for each component.  Just follow the formatting for the
standard components.

There are examples for defining new components in the git repository
in ``examples/new_components/``, including an example of
overriding e.g. ``network.lopf()`` for functionality for
combined-heat-and-power (CHP) plants.
