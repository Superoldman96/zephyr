.. _settings_api:

Settings
########

The settings subsystem gives modules a way to store persistent per-device
configuration and runtime state.  A variety of storage implementations are
provided behind a common API using FCB, NVS, ZMS or a file system.  These
different implementations give the application developer flexibility to select
an appropriate storage medium, and even change it later as needs change.  This
subsystem is used by various Zephyr components and can be used simultaneously by
user applications.

Settings items are stored as key-value pair strings.  By convention,
the keys can be organized by the package and subtree defining the key,
for example the key ``id/serial`` would define the ``serial`` configuration
element for the package ``id``.

Convenience routines are provided for converting a key value to
and from a string type.

For an example of the settings subsystem refer to :zephyr:code-sample:`settings` sample.

.. note::

   As of Zephyr release 4.1 the recommended backends for non-filesystem
   storage are :ref:`NVS <nvs_api>` and :ref:`ZMS <zms_api>`.

Handlers
********

Settings handlers for subtree implement a set of handler functions.
These are registered using a call to :c:func:`settings_register()` for
dynamic handlers or defined using a call to :c:macro:`SETTINGS_STATIC_HANDLER_DEFINE()`
for static handlers.

**h_get**
    This gets called when asking for a settings element value by its name using
    :c:func:`settings_runtime_get()` from the runtime backend.

**h_set**
    This gets called when the value is loaded from persistent storage with
    :c:func:`settings_load()`, or when using :c:func:`settings_runtime_set()` from the
    runtime backend.

**h_commit**
    This gets called after the settings have been loaded in full.
    Sometimes you don't want an individual setting value to take
    effect right away, for example if there are multiple settings
    which are interdependent.

**h_export**
    This gets called to write all current settings. This happens
    when :c:func:`settings_save()` tries to save the settings or transfer to any
    user-implemented back-end.

Settings handlers also have a commit priority ``cprio`` that allows to prioritize
the ``h_commit`` calls. This can be advantageous when e.g. a subsystem initializes
a service that other ``h_commit`` calls depend on.

Settings handlers ``h_commit`` routines are by default initialized with ``cprio = 0``,
initializing a settings handler with a different priority is done using a call to
:c:func:`settings_register_with_cprio()` for dynamic handlers or using a call to
:c:macro:`SETTINGS_STATIC_HANDLER_DEFINE_WITH_CPRIO()` for static handlers. The
specified ``cprio`` value is an integer where lower values mean higher priority.

Backends
********

Backends are meant to load and save data to/from setting handlers, and
implement a set of handler functions. These are registered using a call to
:c:func:`settings_src_register()` for backends that can load data, and/or
:c:func:`settings_dst_register()` for backends that can save data. The current
implementation allows for multiple source backends but only a single destination
backend.

**csi_load**
    This gets called when loading values from persistent storage using
    :c:func:`settings_load()`.

**csi_save**
    This gets called when saving a single setting to persistent storage using
    :c:func:`settings_save_one()`.

**csi_save_start**
    This gets called when starting a save of all current settings using
    :c:func:`settings_save()` or :c:func:`settings_save_subtree()`.

**csi_save_end**
    This gets called after having saved of all current settings using
    :c:func:`settings_save()` or :c:func:`settings_save_subtree()`.

Zephyr Storage Backends
***********************

Zephyr offers the following storage backends:

* Flash Circular Buffer (:kconfig:option:`CONFIG_SETTINGS_FCB`).
* A file in the filesystem (:kconfig:option:`CONFIG_SETTINGS_FILE`).
* Non-Volatile Storage (:kconfig:option:`CONFIG_SETTINGS_NVS`).
* Zephyr Memory Storage (:kconfig:option:`CONFIG_SETTINGS_ZMS`).

You can declare multiple sources for settings; settings from
all of these are restored when :c:func:`settings_load()` is called.

There can be only one target for writing settings; this is where
data is stored when you call :c:func:`settings_save()`, or :c:func:`settings_save_one()`.

FCB read target is registered using :c:func:`settings_fcb_src()`, and write target
using :c:func:`settings_fcb_dst()`. As a side-effect,  :c:func:`settings_fcb_src()`
initializes the FCB area, so it must be called before calling
:c:func:`settings_fcb_dst()`. File read target is registered using
:c:func:`settings_file_src()`, and write target by using :c:func:`settings_file_dst()`.

Non-volatile storage read target is registered using
:c:func:`settings_nvs_src()`, and write target by using
:c:func:`settings_nvs_dst()`.

Zephyr Memory Storage (ZMS) read target is registered using :c:func:`settings_zms_src()`,
and write target is registered using :c:func:`settings_zms_dst()`.

ZMS backend has the particularity of using hash functions to hash the settings
key before storing it to the persistent storage. This implementation implies
that some collisions between key's hashes could occur if a big number of
different keys are stored. This number depends on the selected hash function.

ZMS backend can handle :math:`2^n` maximum collisions where n is defined by
(:kconfig:option:`SETTINGS_ZMS_MAX_COLLISIONS_BITS`).


Storage Location
****************

The FCB, non-volatile storage (NVS) and ZMS backends look for a fixed
partition with label "storage" by default. A different partition can be
selected by setting the ``zephyr,settings-partition`` property of the
chosen node in the devicetree.

The file path used by the file backend to store settings is selected via the
option :kconfig:option:`CONFIG_SETTINGS_FILE_PATH`.

Loading data from persistent storage
************************************

A call to :c:func:`settings_load()` uses an ``h_set`` implementation
to load settings data from storage to volatile memory.
After all data is loaded, the ``h_commit`` handler is issued,
signalling the application that the settings were successfully
retrieved.

Technically FCB and file backends may store some history of the entities.
This means that the newest data entity is stored after any
older existing data entities.
Starting with Zephyr 2.1, the back-end must filter out all old entities and
call the callback with only the newest entity.

Storing data to persistent storage
**********************************

A call to :c:func:`settings_save_one()` uses a backend implementation to store
settings data to the storage medium. A call to :c:func:`settings_save()` uses an
``h_export`` implementation to store different data in one operation using
:c:func:`settings_save_one()`.
A key needs to be covered by a ``h_export`` only if it is supposed to be stored
by :c:func:`settings_save()` call.

For both FCB and file back-end only storage requests with data which
changes most actual key's value are stored, therefore there is no need to check
whether a value changed by the application. Such a storage mechanism implies
that storage can contain multiple value assignments for a key , while only the
last is the current value for the key.

Garbage collection
==================
When storage becomes full (FCB) or consumes too much space (file),
the backend removes non-recent key-value pairs records and unnecessary
key-delete records.

Secure domain settings
**********************
Currently settings doesn't provide scheme of being secure, and non-secure
configuration storage simultaneously for the same instance.
It is recommended that secure domain uses its own settings instance and it might
provide data for non-secure domain using dedicated interface if needed
(case dependent).

Example: Device Configuration
*****************************

This is a simple example, where the settings handler only implements ``h_set``
and ``h_export``. ``h_set`` is called when the value is restored from storage
(or when set initially), and ``h_export`` is used to write the value to
storage thanks to ``storage_func()``. The user can also implement some other
export functionality, for example, writing to the shell console).

.. code-block:: c

    #define DEFAULT_FOO_VAL_VALUE 1

    static int8 foo_val = DEFAULT_FOO_VAL_VALUE;

    static int foo_settings_set(const char *name, size_t len,
                                settings_read_cb read_cb, void *cb_arg)
    {
        const char *next;
        int rc;

        if (settings_name_steq(name, "bar", &next) && !next) {
            if (len != sizeof(foo_val)) {
                return -EINVAL;
            }

            rc = read_cb(cb_arg, &foo_val, sizeof(foo_val));
            if (rc >= 0) {
                /* key-value pair was properly read.
                 * rc contains value length.
                 */
                return 0;
            }
            /* read-out error */
            return rc;
        }

        return -ENOENT;
    }

    static int foo_settings_export(int (*storage_func)(const char *name,
                                                       const void *value,
                                                       size_t val_len))
    {
        return storage_func("foo/bar", &foo_val, sizeof(foo_val));
    }

    struct settings_handler my_conf = {
        .name = "foo",
        .h_set = foo_settings_set,
        .h_export = foo_settings_export
    };

Example: Persist Runtime State
******************************

This is a simple example showing how to persist runtime state. In this example,
only ``h_set`` is defined, which is used when restoring value from
persistent storage.

In this example, the ``main`` function increments ``foo_val``, and then
persists the latest number. When the system restarts, the application calls
:c:func:`settings_load()` while initializing, and ``foo_val`` will continue counting
up from where it was before restart.

.. code-block:: c

    #include <zephyr/kernel.h>
    #include <zephyr/sys/reboot.h>
    #include <zephyr/settings/settings.h>
    #include <zephyr/sys/printk.h>
    #include <inttypes.h>

    #define DEFAULT_FOO_VAL_VALUE 0

    static uint8_t foo_val = DEFAULT_FOO_VAL_VALUE;

    static int foo_settings_set(const char *name, size_t len,
                                settings_read_cb read_cb, void *cb_arg)
    {
        const char *next;
        int rc;

        if (settings_name_steq(name, "bar", &next) && !next) {
            if (len != sizeof(foo_val)) {
                return -EINVAL;
            }

            rc = read_cb(cb_arg, &foo_val, sizeof(foo_val));
            if (rc >= 0) {
                return 0;
            }

            return rc;
        }


        return -ENOENT;
    }

    struct settings_handler my_conf = {
        .name = "foo",
        .h_set = foo_settings_set
    };

    int main(void)
    {
        settings_subsys_init();
        settings_register(&my_conf);
        settings_load();

        foo_val++;
        settings_save_one("foo/bar", &foo_val, sizeof(foo_val));

        printk("foo: %d\n", foo_val);

        k_msleep(1000);
        sys_reboot(SYS_REBOOT_COLD);
    }

Example: Custom Backend Implementation
**************************************

This is a simple example showing how to register a simple custom backend
handler (:kconfig:option:`CONFIG_SETTINGS_CUSTOM`).

.. code-block:: c

    static int settings_custom_load(struct settings_store *cs,
                                    const struct settings_load_arg *arg)
    {
        //...
    }

    static int settings_custom_save(struct settings_store *cs, const char *name,
                                    const char *value, size_t val_len)
    {
        //...
    }

    /* custom backend interface */
    static struct settings_store_itf settings_custom_itf = {
        .csi_load = settings_custom_load,
        .csi_save = settings_custom_save,
    };

    /* custom backend node */
    static struct settings_store settings_custom_store = {
        .cs_itf = &settings_custom_itf
    };

    int settings_backend_init(void)
    {
        /* register custom backend */
        settings_dst_register(&settings_custom_store);
        settings_src_register(&settings_custom_store);
        return 0;
    }

API Reference
*************

The Settings subsystem APIs are provided by :zephyr_file:`include/zephyr/settings/settings.h`.

API for general settings usage
==============================
.. doxygengroup:: settings

API for key-name processing
===========================
.. doxygengroup:: settings_name_proc

API for runtime settings manipulation
=====================================
.. doxygengroup:: settings_rt

API of backend interface
========================
..  doxygengroup:: settings_backend
