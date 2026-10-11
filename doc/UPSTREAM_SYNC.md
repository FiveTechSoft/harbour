# Upstream sync validation

The pending sync merges upstream harbour/core c415d29d9aeb213e3ce6ca6dc41b984fc3908fe4 into the FiveTechSoft fork. Merge ef4bbb1920523d25228322ce7d7644305ffa6cc1 preserves both histories and the compiler's 2026 banner. Source promotion to master remains gated on the fork's five CI workflows and the BCC64, MSVC32 and MSVC64 builds of the final candidate.

## SQLBASE compatibility

The default connection pool remains GLOBAL. The current RDDI_CONNECTION selection is now per-thread. A new thread starts with no selected connection and must explicitly select a global handle before using it. THREAD: connections are local to their owning thread; GLOBAL: connections are shared through the global pool. Existing SQL multithreaded applications that assumed an inherited current connection need to reselect it in each worker.

Directed SQLite multithreaded tests passed for disk and memory databases, 3614 checks each. A separate selection test verified the unprefixed SQLITE3 default is GLOBAL, child selection starts at zero, explicit reassignment succeeds and parent selection is unchanged. This does not establish runtime compatibility for other SQL drivers.

## Clang portability

Clang 23.1.3 adds diagnostics for global variables assigned but never read. The failure also occurred before the upstream merge. Four RDD modules (DBF, DBFCDX, DBFNTX and DBFNSX) held unused RDD-id bookkeeping. GT PCA held an unused cached newline length. Removing that dead state preserves the registration calls and the newline string used for output.

The symbol pointer declared by hbinit.h is different: generated pcode uses the pointer returned by hb_vmProcessSymbols, but native-only registration modules do not. Its declaration is marked optional only for Clang. Registration and pcode execution semantics are unchanged. CI warnings and warnings-as-errors remain enabled; there is no broad warning suppression.

Local checks rebuilt the affected translation units with GCC -W -Wall -Werror, passed 4861 core hbtest checks, and exercised create/append/index/seek/close/reopen operations for the four RDDs. GT PCA compiled separately under the same strict GCC flags. Exact Clang 23.1.3 coverage is provided by the Windows strict CI job, not by these local GCC checks.

## SSL and SIGPIPE

Linux/GCC/OpenSSL 3.0.2 tests survived SIGPIPE and an abruptly closed local TLS peer. The TIP SSL path enabled SIGPIPE ignoring, Close returned zero, the error queue was cleared and the process exited successfully. These checks do not prove runtime behavior on BSD/macOS or every SSL library.

## State

This document records the candidate and test scope, not a completed promotion or a new stable release. Consult the final candidate's CI runs before treating the sync as ready.
