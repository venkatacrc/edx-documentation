.. _driver-setup:

********************
Driver Scripts Setup
********************

The Scripts
***********

Tubular (`edx/tubular on github <https://github.com/edx/tubular>`_) is a
repository of Python 3 scripts designed to plug into various automation
tooling.  Among them are two scripts intended to drive the user retirement
workflow.

``scripts/get_learners_to_retire.py``
    Generates a list of users that are ready for immediate retirement.  Users
    are "ready" after a certain number of days spent in the ``PENDING`` state,
    specified by the ``--cool_off_days`` argument.  Produces an output intended
    for consumption by Jenkins in order to spawn separate downstream builds for
    each user.
``scripts/retire_one_learner.py``
    Retires the user specified by the ``--username`` argument.

All above scripts share a required ``--config_file`` arg which specifies the
driver configuration file for your environment (e.g., production).  This YAML
config contains LMS auth secrets, API URLs, and retirement pipeline stages
specific to that environment.  For example:

.. code-block:: yaml

   client_id: <client ID from previous section>
   client_secret: <client secret from previous section>

   base_urls:
       lms: https://courses.example.com/
       ecommerce: https://ecommerce.example.com/
       credentials: https://credentials.example.com/

   retirement_pipeline:
       - ['RETIRING_EMAIL_LISTS', 'EMAIL_LISTS_COMPLETE', 'LMS', 'retirement_retire_mailings']
       - ['RETIRING_ENROLLMENTS', 'ENROLLMENTS_COMPLETE', 'LMS', 'retirement_unenroll']
       - ['RETIRING_LMS_MISC', 'LMS_MISC_COMPLETE', 'LMS', 'retirement_lms_retire_misc']
       - ['RETIRING_LMS', 'LMS_COMPLETE', 'LMS', 'retirement_lms_retire']

``client_id`` and ``client_secret`` contain the oauth credentials which are
simply copied from the output of the ``create_dot_application`` management
command outlined in :ref:`retirement-service-user`.

The ``base_urls`` section defines the mappings of IDA to base URLs used by the
scripts to construct API URLs.  Only the lms is mandatory here, but if any of
your pipeline states (below) contain API calls to other services, they must
also be present in ``base_urls``.

The ``retirement_pipeline`` section defines the steps, state names, and order
of execution for each environment.  Each item is a list in the form of:

#. start state name
#. end state name
#. IDA to call against (LMS, ECOMMERCE, or CREDENTIALS currently)
#. method name in Tubular's `edx_api.py <https://github.com/edx/tubular/blob/master/tubular/edx_api.py>`_ to call

For example: ``['RETIRING_CREDENTIALS', 'CREDENTIALS_COMPLETE', 'CREDENTIALS',
'retire_learner']`` will set the user's state to ``RETIRING_CREDENTIALS``, call
a pre-instantiated CredentialsApi's retire_learner method, then set the user's
state to ``CREDENTIALS_COMPLETE``.

Examples
--------

Setup your execution environment:

.. code-block:: bash

   git clone https://github.com/edx/tubular.git
   cd tubular
   virtualenv --python=`which python3` venv
   source venv/bin/activate

Generate a list of learners that are ready for retirement, i.e. they each
confirmed account deletion and have been in ``PENDING`` state for the specified
``cool_off_days``.

.. code-block:: bash

   mkdir learners_to_retire
   scripts/get_learners_to_retire.py \
       --config_file=path/to/config.yml \
       --output_dir=learners_to_retire \
       --cool_off_days=5

The ``learners_to_retire`` directory now contains several INI files, each
containing a single line in the form of ``USERNAME=<username-of-learner>``.
Iterate over these files while executing the ``retire_one_learner.py`` on each
learner:

.. code-block:: bash

   scripts/retire_one_learner.py \
       --config_file=path/to/config.yml \
       --username=<username-of-learner-to-retire>


Automation Tooling
******************

At edX we call these scripts from `Jenkins <https://jenkins.io/>`_ jobs on one
of of our internal Jenkins services.  The retirement driver scripts are
intended to be automation tooling agnostic, but they were only fully tested
from Jenkins.

For more information about how we execute these scripts at edX, see the
following wiki articles:

* `GDPR Jenkins Implementation <https://openedx.atlassian.net/wiki/spaces/PLAT/pages/704872737/GDPR+Jenkins+Implementation>`_
* `How to: retirement Jenkins jobs development and testing <https://openedx.atlassian.net/wiki/spaces/PLAT/pages/698221444/How+to+retirement+Jenkins+jobs+development+and+testing>`_

And check out the groovy DSL files used to seed these jobs:

* `platform/jobs/RetirementJobs.groovy in edx/jenkins-job-dsl <https://github.com/edx/jenkins-job-dsl/blob/master/platform/jobs/RetirementJobs.groovy>`_
* `platform/jobs/RetirementJobEdxTriggers.groovy in edx/jenkins-job-dsl <https://github.com/edx/jenkins-job-dsl/blob/master/platform/jobs/RetirementJobEdxTriggers.groovy>`_
