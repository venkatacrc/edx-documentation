
***********************
Implementation Overview
***********************

In the Open edX platform ecosystem, the user experience is enabled by several
services, such as LMS, Studio, ecommerce, credentials, discovery, and more.
Personally Identifiable Identification (PII) about a user exists in many of
these services, so each service containing PII must be requested to
remove/delete/unlink the data for that user in that service.

A centralized process (the *driver* scripts) orchestrates all of these request.
The details of configuring these scripts are outlined in
:ref:`driver-setup`.

The Retirement Workflow
***********************

The user retirement workflow is a configurable pipeline of building-block APIs
that are used to "forget" a user's PII as well as preventing that user from
logging back in, and preventing re-use of their username or email.  Depending
on which third parties a given Open edX site integrates with, the process may
need to involve calling out to external services or generating reports for
later processing (also requiring subsequent destruction of said reports).
Configurability and hackability was a design goal from the beginning, so this
tooling should be able to accommodate a wide range of Open edX sites and custom
use cases.

The workflow is designed to be linear and re-runnable, allowing recovery and
continuation in cases where a particular stage fails.  Each user who has
requested retirement will be individually processed through this workflow, so
multiple users could be in the same state simultaneously.  The LMS is the
authoritative source of information about the state of each user in the
retirement process, and the arbiter of state progressions, via the
UserRetirementStatus model and associated APIs.  It also holds a table of the
states themselves (the RetirementState model), rather than hard-coding the
states.  This was done because we cannot predict all the possible states
required by all members of the Open edX community.

This example state diagram outlines the pathways users follow throughout the
workflow:

.. digraph:: retirement_states_example
   :align: center

      ranksep = "0.3";

      node[fontname=Courier,fontsize=12,shape=box,group=main]
      { rank = same INIT[style=invis] PENDING }
      INIT -> PENDING;
      "..."[shape=none]
      PENDING -> RETIRING_ENROLLMENTS -> ENROLLMENTS_COMPLETE -> RETIRING_FORUMS -> FORUMS_COMPLETE -> "..." -> COMPLETE;

      node[group=""];
      RETIRING_ENROLLMENTS -> ERRORED;
      RETIRING_FORUMS -> ERRORED;
      PENDING -> ABORTED;

      subgraph cluster_terminal_states {
          label = "Terminal States";
          labelloc = b  // put label at bottom
          {rank = same ERRORED COMPLETE ABORTED}
      }

Unless an error occurs internal to the retirement tooling, a user's retirement
state *should* always land in one of the terminal states.  At that point either
their entry should be cleaned up from the UserRetirementStatus table, or if the
state is ``ERRORED`` the administrator needs to take a look (see
:ref:`recovering-from-errored`).

The User Experience
*******************

From the learner's perspective, the vast majority of this process is obscured.
The Account page contains a new section called "Delete My Account" where the
learner may click the "Delete My Account" button and enter their password to
confirm their request.  Subsequently, all of their browser sessions are logged
off, and they become locked out of their account.

An informational email is immediately sent to the learner as an indication of
their account deletion, after which they have a limited amount of time (defined
by the ``--cool_off_days`` argument described in :ref:`driver-setup`) to
contact the administrators and rescind their request.

At this point, the learner's account has been deactivated, but *not* retired.
An entry in the UserRetirementStatus table is added, and their state set to
``PENDING``.

By default, the button is disabled, preventing account deletions.  In order to
open the gate, see :ref:`waffle-switch-for-ux`.

Third Party Auth
----------------

Learners who registered using social auth must first unlink their LMS account
from their third party account(s).  The "Delete My Account" button will be
disabled before they do so; meanwhile they will be instructed to follow this
help center article: `How do I link or unlink my edX account to a social media
account?  <https://support.edx.org/hc/en-us/articles/207206067>`_.
