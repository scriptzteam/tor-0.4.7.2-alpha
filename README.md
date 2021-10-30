# tor-0.4.7.2-alpha

Changes in version 0.4.7.2-alpha - 2021-10-26
  This second alpha release of the 0.4.7.x series adds two major
  features: congestion control (prop324) for network performance, and
  the MiddleOnly flag (prop335) voted by the authorities to pin relays
  to the middle position for various network health reasons. This
  release also fixes numerous bugs.

  The congestion control feature, detailed in proposal 324, still needs
  more work before we can enable it by default. It is currently in its
  testing and tuning phase which means that you should expect more
  0.4.7.x alphas as congestion control gets stabilized and tuned for
  optimal performance. And so, at this release, it can not be used
  without a custom patch.

  o Major features (congestion control):
    - Implement support for flow control over congestion controlled
      circuits. This work comes from proposal 324. Closes ticket 40450.

  o Major features (directory authority):
    - Add a new consensus method to handle MiddleOnly specially. When
      enough authorities are using this method, then any relay tagged
      with the MiddleOnly flag will have its Exit, Guard, HSDir, and
      V2Dir flags automatically cleared, and will have its BadExit flag
      automatically set. Implements part of proposal 335.
    - Authorities can now be configured to label relays as "MiddleOnly".
      When voting for this flag, authorities automatically vote against
      Exit, Guard, HSDir, and V2Dir; and in favor of BadExit. Implements
      part of proposal 335. Based on a patch from Neel Chauhan.

  o Major bugfix (relay, metrics):
    - On the MetricsPort, the DNS error statistics are not reported by
      record type ("record=...") anymore due to a libevent bug
      (https://github.com/libevent/libevent/issues/1219). Fixes bug
      40490; bugfix on 0.4.7.1-alpha.

  o Major bugfixes (relay, overload state):
    - Relays report the general overload state for DNS timeout errors
      only if X% of all DNS queries over Y seconds are errors. Before
      that, it only took 1 timeout to report the overload state which
      was just too low of a threshold. The X and Y values are 1% and 10
      minutes respectively but they are also controlled by consensus
      parameters. Fixes bug 40491; bugfix on 0.4.6.1-alpha.

  o Minor feature (authority, relay):
    - Reject End-Of-Life relays running version 0.4.2.x, 0.4.3.x,
      0.4.4.x and 0.4.5 alphas/rc. Closes ticket 40480.

  o Minor feature (onion service v2):
    - Onion service v2 addresses are now not recognized anymore by tor
      meaning a bad hostname is returned when attempting to pass it on a
      SOCKS connection. No more deprecation log is emitted client side.
      Closes ticket 40476.
    - See https://blog.torproject.org/v2-deprecation-timeline for
      details on how to transition from v2 to v3.

  o Minor features (fallbackdir):
    - Regenerate fallback directories for October 2021. Closes
      ticket 40493.

  o Minor features (logging, heartbeat):
    - When a relay receives a cell that isn't encrypted properly for it,
      but the relay is the last hop on the circuit, the relay now counts
      how many cells of this kind it receives, on how many circuits, and
      reports this information in the log. Previously, we'd log each
      cell at PROTOCOL_WARN level, which is far too verbose to be
      useful. Fixes part of ticket 40400.

  o Minor features (testing):
    - We now have separate fuzzers for the inner layers of v3 onion
      service descriptors, to prevent future bugs like 40392. Closes
      ticket 40488.

  o Minor bugfixes (compilation):
    - Fix compilation error when __NR_time is not defined. Fixes bug
      40465; bugfix on 0.2.5.5-alpha. Patch by Daniel Pinto.

  o Minor bugfixes (dirauth, bandwidth scanner):
    - Add the AuthDirDontVoteOnDirAuthBandwidth dirauth config parameter
      to avoid voting on bandwidth scanner weights to v3 directory
      authorities. Fixes bug 40471; bugfix on 0.2.2.1-alpha. Patch by
      Neel Chauhan.

  o Minor bugfixes (fragile-hardening, sandbox):
    - When building with --enable-fragile-hardening, add or relax Linux
      seccomp rules to allow AddressSanitizer to execute normally if the
      process terminates with the sandbox active. This has the side
      effect of disabling the filtering of file- and directory-open
      requests on most systems and dilutes the effectiveness of the
      sandbox overall, as a wider range of system calls must be
      permitted. Fixes bug 11477; bugfix on 0.2.5.4-alpha.

  o Minor bugfixes (logging):
    - If a channel has never received or transmitted a cell, or seen a
      client, do not calculate time diffs against 1/1/1970 but log a
      better prettier message. Fixes bug 40182; bugfix on 0.2.4.4.

  o Minor bugfixes (onion service):
    - Fix a warning BUG that would occur often on heavily loaded onion
      service leading to filling the logs with useless warnings. Fixes
      bug 34083; bugfix on 0.3.2.1-alpha.

  o Minor bugfix (CI, onion service):
    - Exclude onion service version 2 Stem tests in our CI. Fixes bug 40500;
      bugfix on 0.3.2.1-alpha.

  o Minor bugfixes (onion service, config):
    - Fix a memory leak for a small config line string that could occur
      if the onion service failed to be configured from file properly.
      Fixes bug 40484; bugfix on 0.3.2.1-alpha.

  o Minor bugfixes (onion service, TROVE-2021-008):
    - Only log v2 access attempts once total, in order to not pollute
      the logs with warnings and to avoid recording the times on disk
      when v2 access was attempted. Note that the onion address was
      _never_ logged. This counts as a Low-severity security issue.
      Fixes bug 40474; bugfix on 0.4.5.8.
    - Note that due to #40476 which removes v2 support entirely, this
      log line is not emitted anymore. We still mention this in the
      changelog because it is a Low-severity TROVE.

  o Minor bugfixes (usability):
    - Do not log "RENDEZVOUS1 cell with unrecognized rendezvous cookie"
      at LOG_PROTOCOL_WARN; instead log it at DEBUG. This warning can
      happen naturally if a client gives up on a rendezvous circuit
      after sending INTRODUCE1. Fixes part of bug 40400; bugfix
      on 0.1.1.13-alpha.
    - Do not log "circuit_receive_relay_cell failed" at
      LOG_PROTOCOL_WARN; instead log it at DEBUG. In every case where we
      would want to log this as a protocol warning, we are already
      logging another warning from inside circuit_receive_relay_cell.
      Fixes part of bug 40400; bugfix on 0.1.1.9-alpha.

  o Code simplification and refactoring:
    - Lower the official maximum for "guard-extreme-restriction-percent"
      to 100. This has no effect on when the guard code will generate a
      warning, but it makes the intent of the option clearer. Fixes bug
      40486; bugfix on 0.3.0.1-alpha.

  o Testing:
    - Add unit tests for the Linux seccomp sandbox. Resolves
      issue 16803.

  o Code simplification and refactoring (rust):
    - Remove Rust support and its associated code. It is unsupported and
      Rust focus should be shifted to arti. Closes ticket 40469.

  o Testing (CI, chutney):
    - Bump the data size that chutney transmits to 5MBytes in order to
      trigger the flow control and congestion window code. Closes
      ticket 40485.
