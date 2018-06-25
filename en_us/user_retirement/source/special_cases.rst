.. _handling-special-cases:

**********************
Handling Special Cases
**********************

.. _recovering-from-errored:

Recovering from ERRORED
***********************

If a retirement API indicates failure (4xx or 5xx status code), the driver
immediately sets the user's state to ``ERRORED``.  You may take this
opportunity to check the ``responses`` field in the user's row in
user_api_userretirementstatus (User Retirement Status) for any relevant logging
to help debug the error.  Once the issue is resolved, you need to manually set
the user's ``current_state`` to the state immediately prior to the state which
should be retried.  This can be done via Django admin.

.. digraph:: retirement_states_example
   :align: center

      //rankdir=LR;  // Rank Direction Left to Right
      ranksep = "0.3";

      edge[color=grey]

      node[fontname=Courier,fontsize=12,shape=box,group=main]
      { rank = same INIT[style=invis] PENDING }
      {
          edge[style=bold,color=black]
          INIT -> PENDING;
          "..."[shape=none]
          PENDING -> RETIRING_ENROLLMENTS -> ENROLLMENTS_COMPLETE -> RETIRING_FORUMS;
      }
      RETIRING_FORUMS -> FORUMS_COMPLETE -> "..." -> COMPLETE

      node[group=""];
      RETIRING_ENROLLMENTS -> ERRORED;
      RETIRING_FORUMS -> ERRORED[style=bold,color=black];
      PENDING -> ABORTED;

      subgraph cluster_terminal_states {
          label = "Terminal States";
          labelloc = b  // put label at bottom
          {rank = same ERRORED COMPLETE ABORTED}
      }

      ERRORED -> ENROLLMENTS_COMPLETE[style="bold,dashed",color=black,label=" via django\nadmin"]

Now, the retirement driver scripts will automatically resume this user's
retirement the next time they are executed.

Re-running some or all states
*****************************

If you decide you want to re-run all retirements from the beginning, simply set
``current_state`` to ``PENDING`` for all retirements with ``current_state`` ==
``COMPLETE``.  This would be useful in the case where a new stage is added
after running all retirements (but before the retirement queue is cleaned up),
and you want to run all the retirements through the new stage.  Or, perhaps you
were developing a stage/API that didn't work correctly but still indicated
success, so the pipeline progressed all users into ``COMPLETED``.  Retirement
APIs are designed to be idempotent, so this should be a no-op for stages
already run for a given user.
