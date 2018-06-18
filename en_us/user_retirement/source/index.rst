.. _document index:

#################################################
User Retirement Guide for Open edX Administrators
#################################################

GDPR (or the EU General Data Protection Regulation) is a sweeping change to
privacy laws intended to change the way that business think about and handle
Personally Identifiable Information (PII) data.  In order to comply with this
law, edX has implemented APIs and tooling for "retiring" users.  Once
configured, the user retirement tooling can automatically erase PII for a given
user from internal systems (i.e. lms, forums, credentials, and other IDAs) and
external (e.g. third party marketing services).

The following sections are not only intended for instructing admins to perform
the basic setup, but also to offer some insight into the implementation of the
retirement tooling in order to help the Open edX community build additional
APIs and states that meet their special needs. Custom code, plugins, packages
or XBlocks might store PII, but this tooling will not magically find and
cleanup that PII.

.. toctree::
    :numbered:
    :maxdepth: 2

    implementation_overview
    service_setup
    driver_setup
    special_cases

