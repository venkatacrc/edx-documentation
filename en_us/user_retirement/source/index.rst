.. _document index:

#################################################
User Retirement Guide for Open edX Administrators
#################################################

The European Union General Data Protection Regulation (GDPR) is a sweeping
change to privacy laws intended to change the way that business think about
and handle Personally Identifiable Information (PII) data. As a step toward
enabling Open edX to comply with GDPR, edX has implemented APIs and tooling
that enables Open edX instances to retire registered users. When you implement
this user retirement feature, your Open edX instance can automatically erase
PII for a given user from systems that are internal to Open edX (for example,
the LMS, forums, credentials, and other independently deployable applications
(IDAs)), as well as external systems, such as third party marketing services.

This guide is not only intended for instructing Open edX admins to perform
the basic setup, but also to offer some insight into the implementation of the
user retirement feature in order to help the Open edX community build
additional APIs and states that meet their special needs. Custom code,
plugins, packages, or XBlocks in your Open edX instance might store PII, but
this tooling will not magically find and clean up that PII.

.. toctree::
    :numbered:
    :maxdepth: 2

    implementation_overview
    service_setup
    driver_setup
    special_cases

