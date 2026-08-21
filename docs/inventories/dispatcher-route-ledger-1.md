# Dispatcher route ledger A–O

Baseline: `775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`. See [index and method](dispatcher-route-ledger-index.md).

| Generated spelling(s) | Nested router | Exact handler | Source fragment | Proposed disposition |
|---|---|---|---|---|
| `__async-dispatch-test` | test-only (`cfg(test)`) | `Async(run_async_dispatch_test)` | `core_impl/dispatcher.rs` `DISPATCH_01` | **COMPAT/UNRESOLVED** |
| `about` | — | `Sync(run_about_command)` | `core_impl/about.rs` `DISPATCH_39` | **COMPAT/UNRESOLVED** |
| `absorb` | — | `Sync(run_absorb_command)` | `core_impl/absorb.rs` `DISPATCH_60` | **COMPAT/UNRESOLVED** |
| `activity` | — | `Sync(activity_run_command)` | `core_impl/activity.rs` `DISPATCH_107` | **COMPAT/UNRESOLVED** |
| `agents` / `agent` | [`agents`](dispatcher-route-ledger-index.md#nested-agents) | `Sync(agents_run_command)` | `core_impl/agents.rs` `DISPATCH_70` | **COMPAT/UNRESOLVED** |
| `alive` | — | `Sync(alive_run_command)` | `core_impl/tmux_alive_inspect.rs` `DISPATCH_268` | **RFC0002-TMUX** |
| `archive` | — | `Sync(run_archive_command)` | `core_impl/archive.rs` `DISPATCH_45` | **COMPAT/UNRESOLVED** |
| `artifacts` / `artifact` | [`artifacts`](dispatcher-route-ledger-index.md#nested-artifacts) | `Sync(run_artifacts_command)` | `core_impl/artifacts.rs` `DISPATCH_71` | **EXTERNAL-963** |
| `assign` | — | `Async(run_assign_async)` | `core_impl/assign.rs` `DISPATCH_48` | **COMPAT/UNRESOLVED** |
| `attach` / `a` | — | `Sync(attach_run_command)` | `core_impl/attach.rs` `DISPATCH_111` | **KERNEL5** |
| `attach-ssh` | — | `Sync(attachssh_run_command)` | `core_impl/attach_ssh.rs` `DISPATCH_115` | **COMPAT/UNRESOLVED** |
| `audit` | — | `Sync(run_audit_command)` | `core_impl/audit.rs` `DISPATCH_69` | **HOST-ADMIN** |
| `auth` | [`auth`](dispatcher-route-ledger-index.md#nested-auth) | `Sync(auth_run_command)` | `core_impl/auth.rs` `DISPATCH_96` | **HOST-ADMIN** |
| `auto-pair-proof` | — | `Sync(run_auto_pair_proof_plan)` | `core_impl/pair_consent.rs` `DISPATCH_315` | **COMPAT/UNRESOLVED** |
| `auto-wake` | — | `Sync(run_auto_wake_plan)` | `core_impl/dispatcher.rs` `DISPATCH_01` | **COMPAT/UNRESOLVED** |
| `awake` | — | `Sync(awake_run_command)` | `core_impl/workspace_scaffold_commands.rs` `DISPATCH_93` | **COMPAT/UNRESOLVED** |
| `awaken` | — | `Sync(run_awaken_command)` | `core_impl/awaken.rs` `DISPATCH_54` | **COMPAT/UNRESOLVED** |
| `bg` | [`bg`](dispatcher-route-ledger-index.md#nested-bg) | `Sync(bg_run_command)` | `core_impl/background_jobs.rs` `DISPATCH_88` | **COMPAT/UNRESOLVED** |
| `bind-host` | — | `Sync(run_bind_host_plan)` | `core_impl/plugin_manifest_bind_host.rs` `DISPATCH_302` | **HOST-ADMIN** |
| `break` | — | `Sync(run_break_command)` | `core_impl/tmux_break.rs` `DISPATCH_280` | **RFC0002-TMUX** |
| `bring` / `b` | — | `Sync(run_bring_plan)` | `core_impl/session_list_plan.rs` `DISPATCH_304` | **EXTERNAL-963** |
| `bud` / `buddy` | — | `Sync(bud_run_command)` | `core_impl/buddy_workspace.rs` `DISPATCH_121` | **COMPAT/UNRESOLVED** |
| `calver` | — | `Sync(run_calver_plan)` | `core_impl/worktree_window.rs` `DISPATCH_310` | **COMPAT/UNRESOLVED** |
| `capture` | — | `Sync(capture_run_command)` | `core_impl/capture.rs` `DISPATCH_77` | **RFC0002-TMUX** |
| `census` | — | `Sync(census_run_command)` | `core_impl/census.rs` `DISPATCH_380` | **COMPAT/UNRESOLVED** |
| `channel` | [`channel`](dispatcher-route-ledger-index.md#nested-channel) | `Sync(channel_run_command)` | `core_impl/channel.rs` `DISPATCH_120` | **COMPAT/UNRESOLVED** |
| `check` | [`check`](dispatcher-route-ledger-index.md#nested-check) | `Sync(run_check_command)` | `core_impl/check_tools.rs` `DISPATCH_137` | **HOST-ADMIN** |
| `cleanup` | — | `Sync(run_cleanup_fail_closed_command)` | `core_impl/learn_project.rs` `DISPATCH_291` | **COMPAT/UNRESOLVED** |
| `codex` | [`codex`](dispatcher-route-ledger-index.md#nested-codex) | `Sync(codex_run_command)` | `core_impl/codex_accounts.rs` `DISPATCH_273` | **EXTERNAL-963** |
| `commands` | — | `Sync(commands_handler)` | `core_impl/dispatcher.rs` `DISPATCH_01` | **HOST-ADMIN** |
| `completions` | [`completions`](dispatcher-route-ledger-index.md#nested-completions) | `Sync(completions_run_command)` | `core_impl/completions.rs` `DISPATCH_99` | **HOST-ADMIN** |
| `config` | [`config`](dispatcher-route-ledger-index.md#nested-config) | `Sync(config_run_command)` | `core_impl/config.rs` `DISPATCH_136` | **HOST-ADMIN** |
| `consent` | [`consent`](dispatcher-route-ledger-index.md#nested-consent) | `Sync(run_consent_command_135)` | `core_impl/consent.rs` `DISPATCH_142` | **HOST-ADMIN** |
| `consent-approval` | — | `Sync(run_consent_approval_plan)` | `core_impl/consent_request.rs` `DISPATCH_316` | **HOST-ADMIN** |
| `consent-cleanup` | — | `Sync(run_consent_cleanup_plan)` | `core_impl/consent_trust.rs` `DISPATCH_318` | **HOST-ADMIN** |
| `consent-constants` | — | `Sync(run_consent_constants_plan)` | `core_impl/consent_request.rs` `DISPATCH_316` | **HOST-ADMIN** |
| `consent-expiry` | — | `Sync(run_consent_expiry_plan)` | `core_impl/consent_store.rs` `DISPATCH_317` | **HOST-ADMIN** |
| `consent-pending-read` | — | `Sync(run_consent_pending_read_plan)` | `core_impl/consent_trust.rs` `DISPATCH_318` | **HOST-ADMIN** |
| `consent-pending-status` | — | `Sync(run_consent_pending_status_plan)` | `core_impl/consent_trust.rs` `DISPATCH_318` | **HOST-ADMIN** |
| `consent-pin` | — | `Sync(run_consent_pin_plan)` | `core_impl/pair_consent.rs` `DISPATCH_315` | **COMPAT/UNRESOLVED** |
| `consent-request` | — | `Sync(run_consent_request_plan)` | `core_impl/consent_request.rs` `DISPATCH_316` | **HOST-ADMIN** |
| `consent-store` | — | `Sync(run_consent_store_plan)` | `core_impl/consent_store.rs` `DISPATCH_317` | **HOST-ADMIN** |
| `consent-trust-check` | — | `Sync(run_consent_trust_check_plan)` | `core_impl/consent_trust.rs` `DISPATCH_318` | **HOST-ADMIN** |
| `consent-trust-revoke` | — | `Sync(run_consent_trust_revoke_plan)` | `core_impl/consent_trust.rs` `DISPATCH_318` | **HOST-ADMIN** |
| `cross-team-queue` | — | `Sync(ctq_run_command)` | `core_impl/cross_team_queue.rs` `DISPATCH_143` | **COMPAT/UNRESOLVED** |
| `discover` | — | `Sync(run_discover_plan)` | `core_impl/discover_plan.rs` `DISPATCH_312` | **COMPAT/UNRESOLVED** |
| `doctor` | — | `Sync(run_doctor_command)` | `core_impl/doctor.rs` `DISPATCH_65` | **HOST-ADMIN** |
| `done` / `finish` | — | `Sync(run_done_command)` | `core_impl/worktree_finish.rs` `DISPATCH_57` | **COMPAT/UNRESOLVED** |
| `federation` | [`federation`](dispatcher-route-ledger-index.md#nested-federation) | `Sync(federation_run_command)` | `core_impl/federation.rs` `DISPATCH_141` | **COMPAT/UNRESOLVED** |
| `federation-health` | — | `Sync(run_federation_health_plan)` | `core_impl/federation_identity.rs` `DISPATCH_313` | **COMPAT/UNRESOLVED** |
| `federation-identity` | — | `Sync(run_federation_identity_plan)` | `core_impl/federation_identity.rs` `DISPATCH_313` | **COMPAT/UNRESOLVED** |
| `federation-sync` | — | `Sync(run_federation_sync_plan)` | `core_impl/peer_sources.rs` `DISPATCH_314` | **COMPAT/UNRESOLVED** |
| `feed` | [`feed`](dispatcher-route-ledger-index.md#nested-feed) | `Sync(run_feed_plan)` | `core_impl/bind_feed_fuzzy_plan.rs` `DISPATCH_306` | **COMPAT/UNRESOLVED** |
| `find` | — | `Sync(find_run_command)` | `core_impl/find.rs` `DISPATCH_110` | **COMPAT/UNRESOLVED** |
| `fleet` | [`fleet`](dispatcher-route-ledger-index.md#nested-fleet) | `Sync(run_fleet_command)` | `core_impl/fleet.rs` `DISPATCH_61` | **COMPAT/UNRESOLVED** |
| `focus` | — | `Sync(focus_run_command)` | `core_impl/tmux_resize_focus.rs` `DISPATCH_328` | **RFC0002-TMUX** |
| `footer` | — | `Sync(footer_run_command263)` | `core_impl/footer.rs` `DISPATCH_263` | **COMPAT/UNRESOLVED** |
| `forget` | — | `Sync(run_forget_command)` | `core_impl/forget.rs` `DISPATCH_56` | **COMPAT/UNRESOLVED** |
| `forward-error` | — | `Async(forwarderror_async)` | `core_impl/forward_error.rs` `DISPATCH_92` | **COMPAT/UNRESOLVED** |
| `fuzzy` | [`fuzzy`](dispatcher-route-ledger-index.md#nested-fuzzy) | `Sync(run_fuzzy_plan)` | `core_impl/bind_feed_fuzzy_plan.rs` `DISPATCH_306` | **COMPAT/UNRESOLVED** |
| `gather` | — | `Sync(run_team_gather_command)` | `core_impl/team_gather_scatter.rs` `DISPATCH_329` | **EXTERNAL-963** |
| `health` | — | `Async(run_health_async)` | `core_impl/send_federation.rs` `DISPATCH_307` | **COMPAT/UNRESOLVED** |
| `--help` / `-h` / `help` | — | `Sync(usage_handler)` | `core_impl/dispatcher.rs` `DISPATCH_01` | **HOST-ADMIN** |
| `hey` | — | `Async(run_hey_async)` | `core_impl/send_federation.rs` `DISPATCH_307` | **KERNEL5** |
| `identity` | [`identity`](dispatcher-route-ledger-index.md#nested-identity) | `Sync(run_identity_plan)` | `core_impl/fuzzy_identity.rs` `DISPATCH_309` | **COMPAT/UNRESOLVED** |
| `inbox` | [`inbox`](dispatcher-route-ledger-index.md#nested-inbox) | `Async(run_inbox_command)` | `core_impl/inbox.rs` `DISPATCH_62` | **COMPAT/UNRESOLVED** |
| `init` | — | `Sync(init_run_command)` | `core_impl/config_init.rs` `DISPATCH_105` | **HOST-ADMIN** |
| `inspect` | — | `Sync(inspect_run_command)` | `core_impl/tmux_alive_inspect.rs` `DISPATCH_268` | **RFC0002-TMUX** |
| `join` | — | `Sync(join_run_command)` | `core_impl/tmux_join.rs` `DISPATCH_264` | **RFC0002-TMUX** |
| `kill` | — | `Sync(kill_run_command)` | `core_impl/kill.rs` `DISPATCH_78` | **RFC0002-TMUX** |
| `learn` | — | `Sync(run_learn_command)` | `core_impl/learn_project.rs` `DISPATCH_291` | **COMPAT/UNRESOLVED** |
| `locate` | — | `Sync(run_locate_command)` | `core_impl/locate.rs` `DISPATCH_41` | **COMPAT/UNRESOLVED** |
| `ls` | — | `Async(run_ls_plan_async)` | `core_impl/fleet_list.rs` `DISPATCH_151` | **COMPAT/UNRESOLVED** |
| `messages` | — | `Async(run_messages_async)` | `core_impl/serve_daemon.rs` `DISPATCH_152` | **COMPAT/UNRESOLVED** |
| `more` | [`more`](dispatcher-route-ledger-index.md#nested-more) | `Sync(run_more_command)` | `core_impl/more.rs` `DISPATCH_324` | **EXTERNAL-963** |
| `mv` | — | `Sync(mv_run_command)` | `core_impl/tmux_handover.rs` `DISPATCH_81` | **RFC0002-TMUX** |
| `new` | — | `Sync(new_run_command)` | `core_impl/workspace_scaffold_commands.rs` `DISPATCH_93` | **COMPAT/UNRESOLVED** |
| `normalize` | — | `Sync(run_normalize_plan)` | `core_impl/resolve_plan.rs` `DISPATCH_308` | **COMPAT/UNRESOLVED** |
| `notify` | — | `Async(run_notify_async)` | `core_impl/notify.rs` `DISPATCH_83` | **COMPAT/UNRESOLVED** |
| `on` | — | `Sync(run_on_command)` | `core_impl/trigger_registration.rs` `DISPATCH_42` | **COMPAT/UNRESOLVED** |
| `oracle` / `oracles` | [`oracle`](dispatcher-route-ledger-index.md#nested-oracle) | `Sync(run_oracle_command)` | `core_impl/oracle.rs` `DISPATCH_63` | **COMPAT/UNRESOLVED** |
| `oracle-recruit` | — | `Sync(run_oracle_recruit_command)` | `core_impl/oracle_recruit.rs` `DISPATCH_332` | **COMPAT/UNRESOLVED** |
| `oracle-skills` | — | `Sync(oracle_skills_run_command)` | `core_impl/oracle_skills.rs` `DISPATCH_200` | **COMPAT/UNRESOLVED** |
| `oracle-workon` | — | `Sync(oracleworkon_run_command)` | `core_impl/oracle_workon.rs` `DISPATCH_90` | **COMPAT/UNRESOLVED** |
| `overview` | — | `Sync(run_overview_command)` | `core_impl/overview.rs` `DISPATCH_40` | **COMPAT/UNRESOLVED** |

Rows: **85**; source spellings covered: **94**.
