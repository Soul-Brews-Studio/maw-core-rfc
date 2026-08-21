# Dispatcher route ledger P–Z

Baseline: `775c709b67add2a69c7c2dd3ecee18bbb5fcdb6e`. See [index and method](dispatcher-route-ledger-index.md).

| Generated spelling(s) | Nested router | Exact handler | Source fragment | Proposed disposition |
|---|---|---|---|---|
| `pair` | [`pair`](dispatcher-route-ledger-index.md#nested-pair) | `Sync(pair_run_command)` | `core_impl/pair.rs` `DISPATCH_97` | **COMPAT/UNRESOLVED** |
| `pair-api` | [`pair-api`](dispatcher-route-ledger-index.md#nested-pair-api) | `Sync(run_pair_api_plan)` | `core_impl/pair_api.rs` `DISPATCH_320` | **COMPAT/UNRESOLVED** |
| `pair-api-auto` | — | `Sync(run_pair_api_auto_plan)` | `core_impl/pair_api_auto.rs` `DISPATCH_321` | **COMPAT/UNRESOLVED** |
| `pair-code` | — | `Sync(run_pair_code_plan)` | `core_impl/pair_code.rs` `DISPATCH_319` | **COMPAT/UNRESOLVED** |
| `pair-code-store` | [`pair-code-store`](dispatcher-route-ledger-index.md#nested-pair-code-store) | `Sync(run_pair_code_store_plan)` | `core_impl/pair_code.rs` `DISPATCH_319` | **COMPAT/UNRESOLVED** |
| `pane` | [`pane`](dispatcher-route-ledger-index.md#nested-pane) | `Sync(run_pane_command)` | `core_impl/pane_swap.rs` `DISPATCH_73` | **RFC0002-TMUX** |
| `panes` | — | `Sync(run_panes_command)` | `core_impl/tmux_panes.rs` `DISPATCH_76` | **RFC0002-TMUX** |
| `park` | — | `Sync(run_park_command)` | `core_impl/learn_project.rs` `DISPATCH_291` | **COMPAT/UNRESOLVED** |
| `peek` | — | `Sync(peek_run_command)` | `core_impl/tmux_peek.rs` `DISPATCH_134` | **KERNEL5** |
| `peer-probe` | [`peer-probe`](dispatcher-route-ledger-index.md#nested-peer-probe) | `Sync(run_peer_probe_plan)` | `core_impl/peer_probe.rs` `DISPATCH_322` | **COMPAT/UNRESOLVED** |
| `peer-sources` | — | `Sync(run_peer_sources_plan)` | `core_impl/peer_sources.rs` `DISPATCH_314` | **COMPAT/UNRESOLVED** |
| `peers` / `peer` | [`peers`](dispatcher-route-ledger-index.md#nested-peers) | `Sync(peers_run_command)` | `core_impl/peers.rs` `DISPATCH_104` | **COMPAT/UNRESOLVED** |
| `ping` | — | `Sync(ping_run_command)` | `core_impl/ping.rs` `DISPATCH_138` | **COMPAT/UNRESOLVED** |
| `plugin` | [`plugin`](dispatcher-route-ledger-index.md#nested-plugin) | `Sync(plugin_run_command)` | `core_impl/plugin.rs` `DISPATCH_102` | **HOST-ADMIN** |
| `plugin-artifact` | [`plugin-artifact`](dispatcher-route-ledger-index.md#nested-plugin-artifact) | `Sync(pluginartifact_run_command)` | `core_impl/plugin_artifact.rs` `DISPATCH_288` | **HOST-ADMIN** |
| `plugin-manifest` | [`plugin-manifest`](dispatcher-route-ledger-index.md#nested-plugin-manifest) | `Sync(run_plugin_manifest_plan)` | `core_impl/plugin_manifest.rs` `DISPATCH_287` | **HOST-ADMIN** |
| `plugin-scaffold` | [`plugin-scaffold`](dispatcher-route-ledger-index.md#nested-plugin-scaffold) | `Sync(run_plugin_scaffold_plan)` | `core_impl/xdg_plugin_scaffold_plan.rs` `DISPATCH_301` | **HOST-ADMIN** |
| `plugins` | [`plugins`](dispatcher-route-ledger-index.md#nested-plugins) | `Sync(plugins_run_command)` | `core_impl/plugins.rs` `DISPATCH_101` | **HOST-ADMIN** |
| `policy` / `plugin-policy` | [`policy`](dispatcher-route-ledger-index.md#nested-policy) | `Sync(run_policy_plan)` | `core_impl/transport_policy.rs` `DISPATCH_323` | **HOST-ADMIN** |
| `pr` | — | `Sync(run_pr_command)` | `core_impl/github_pull_request.rs` `DISPATCH_58` | **COMPAT/UNRESOLVED** |
| `preflight` | — | `Sync(preflight_run_command)` | `core_impl/workspace_scaffold_commands.rs` `DISPATCH_93` | **COMPAT/UNRESOLVED** |
| `profile` | [`profile`](dispatcher-route-ledger-index.md#nested-profile) | `Sync(profile_run_command)` | `core_impl/profile.rs` `DISPATCH_135` | **COMPAT/UNRESOLVED** |
| `project` | [`project`](dispatcher-route-ledger-index.md#nested-project) | `Sync(run_project_command)` | `core_impl/learn_project.rs` `DISPATCH_291` | **COMPAT/UNRESOLVED** |
| `promote` | — | `Sync(promote_run_command)` | `core_impl/workspace_scaffold_commands.rs` `DISPATCH_93` | **COMPAT/UNRESOLVED** |
| `recent-hello` | — | `Sync(run_recent_hello_plan)` | `core_impl/pair_code.rs` `DISPATCH_319` | **COMPAT/UNRESOLVED** |
| `reindex-gpu` | — | `Async(reindex_async_native)` | `core_impl/reindex_gpu.rs` `DISPATCH_290` | **COMPAT/UNRESOLVED** |
| `rename` | — | `Sync(run_rename_command)` | `core_impl/rename.rs` `DISPATCH_38` | **RFC0002-TMUX** |
| `rename-pane` | — | `Sync(rename_pane_run_command)` | `core_impl/tmux_resize_focus.rs` `DISPATCH_328` | **RFC0002-TMUX** |
| `reply` / `rp` | — | `Async(run_reply_async)` | `core_impl/send_federation.rs` `DISPATCH_307` | **COMPAT/UNRESOLVED** |
| `resize` | — | `Sync(resize_run_command)` | `core_impl/tmux_resize_focus.rs` `DISPATCH_328` | **RFC0002-TMUX** |
| `resolve` | — | `Sync(run_resolve_plan)` | `core_impl/resolve_plan.rs` `DISPATCH_308` | **COMPAT/UNRESOLVED** |
| `restart` / `reboot` | — | `Sync(run_restart_command)` | `core_impl/restart.rs` `DISPATCH_59` | **COMPAT/UNRESOLVED** |
| `resume` | — | `Sync(resume_run_command)` | `core_impl/resume.rs` `DISPATCH_87` | **COMPAT/UNRESOLVED** |
| `reunion` | — | `Sync(reunion_run_command)` | `core_impl/reunion.rs` `DISPATCH_89` | **COMPAT/UNRESOLVED** |
| `route` | — | `Sync(run_route_plan)` | `core_impl/route_plan.rs` `DISPATCH_311` | **HOST-ADMIN** |
| `run` | — | `Sync(run_native_command)` | `core_impl/target_command_run.rs` `DISPATCH_91` | **COMPAT/UNRESOLVED** |
| `scaffold` | — | `Sync(scaffold_run_command)` | `core_impl/workspace_scaffold_commands.rs` `DISPATCH_93` | **COMPAT/UNRESOLVED** |
| `scatter` | — | `Sync(run_team_scatter_command)` | `core_impl/team_gather_scatter.rs` `DISPATCH_329` | **EXTERNAL-963** |
| `schedule` | [`schedule`](dispatcher-route-ledger-index.md#nested-schedule) | `Sync(schedule_run_command334)` | `core_impl/schedule.rs` `DISPATCH_334` | **COMPAT/UNRESOLVED** |
| `scope` | [`scope`](dispatcher-route-ledger-index.md#nested-scope) | `Sync(scope_run_command)` | `core_impl/scope.rs` `DISPATCH_109` | **HOST-ADMIN** |
| `scout` / `zenoh-scout` | — | `Async(zenohscout_async_native)` | `core_impl/zenoh_scout.rs` `DISPATCH_103` | **COMPAT/UNRESOLVED** |
| `send` | — | `Async(run_send_async)` | `core_impl/send_federation.rs` `DISPATCH_307` | **COMPAT/UNRESOLVED** |
| `send-enter` | — | `Sync(run_send_enter_command)` | `core_impl/tmux_attach.rs` `DISPATCH_305` | **RFC0002-TMUX** |
| `send-escape` | — | `Sync(run_send_escape_command)` | `core_impl/tmux_attach.rs` `DISPATCH_305` | **RFC0002-TMUX** |
| `send-key` | — | `Sync(run_send_key_command)` | `core_impl/tmux_attach.rs` `DISPATCH_305` | **RFC0002-TMUX** |
| `send-text` | — | `Sync(sendtext_run_command)` | `core_impl/send_text.rs` `DISPATCH_84` | **RFC0002-TMUX** |
| `serve` | [`serve`](dispatcher-route-ledger-index.md#nested-serve) | `Async(run_serve_async)` | `core_impl/serve_daemon.rs` `DISPATCH_152` | **RFC0003-SERVE** |
| `serve-identity` | — | `Sync(serveidentity_command)` | `core_impl/serve_identity.rs` `DISPATCH_95` | **RFC0003-SERVE** |
| `serve-peer-startup-warnings` | — | `Sync(servepeerstartupwarnings_run_command)` | `core_impl/serve_peer_warnings.rs` `DISPATCH_139` | **RFC0003-SERVE** |
| `session` | — | `Sync(session_run_command)` | `core_impl/session.rs` `DISPATCH_75` | **COMPAT/UNRESOLVED** |
| `setup` | [`setup`](dispatcher-route-ledger-index.md#nested-setup) | `Sync(run_setup_command)` | `core_impl/setup.rs` `DISPATCH_67` | **COMPAT/UNRESOLVED** |
| `shellenv` | — | `Sync(shellenv_run_command)` | `core_impl/shellenv.rs` `DISPATCH_133` | **COMPAT/UNRESOLVED** |
| `signals` | — | `Sync(run_signals_command)` | `core_impl/signals.rs` `DISPATCH_46` | **COMPAT/UNRESOLVED** |
| `sleep` | — | `Sync(sleep_run_command)` | `core_impl/sleep.rs` `DISPATCH_118` | **COMPAT/UNRESOLVED** |
| `snapshots` | [`snapshots`](dispatcher-route-ledger-index.md#nested-snapshots) | `Sync(snapshots_run_command)` | `core_impl/workspace_scaffold_commands.rs` `DISPATCH_93` | **COMPAT/UNRESOLVED** |
| `split` | — | `Sync(split_run_command)` | `core_impl/split.rs` `DISPATCH_113` | **EXTERNAL-963** |
| `split-policy` | — | `Sync(run_split_policy_plan)` | `core_impl/peer_probe.rs` `DISPATCH_322` | **COMPAT/UNRESOLVED** |
| `squad` | [`squad`](dispatcher-route-ledger-index.md#nested-squad) | `Sync(run_squad_command)` | `core_impl/squad.rs` `DISPATCH_333` | **COMPAT/UNRESOLVED** |
| `stop` / `rest` | — | `Sync(stop_run_command)` | `core_impl/fleet_stop.rs` `DISPATCH_86` | **COMPAT/UNRESOLVED** |
| `swap` | — | `Sync(swap_run_command)` | `core_impl/tmux_swap.rs` `DISPATCH_266` | **RFC0002-TMUX** |
| `swarm` | — | `Sync(swarm_run_command)` | `core_impl/swarm.rs` `DISPATCH_117` | **EXTERNAL-963** |
| `tab` | [`tab`](dispatcher-route-ledger-index.md#nested-tab) | `Sync(run_tab_command)` | `core_impl/tmux_tab.rs` `DISPATCH_43` | **RFC0002-TMUX** |
| `tag` | — | `Sync(tag_run_command)` | `core_impl/tmux_tag.rs` `DISPATCH_82` | **RFC0002-TMUX** |
| `take` / `handover` | — | `Sync(take_run_command)` | `core_impl/tmux_handover.rs` `DISPATCH_81` | **RFC0002-TMUX** |
| `talk-to` / `talkto` / `talk` | — | `Async(run_talkto_async)` | `core_impl/talk_to.rs` `DISPATCH_85` | **COMPAT/UNRESOLVED** |
| `team` / `t` | [`team`](dispatcher-route-ledger-index.md#nested-team) | `Sync(team_run_command)` | `core_impl/team_enter.rs` `DISPATCH_240` | **EXTERNAL-963** |
| `tmux` | [`tmux`](dispatcher-route-ledger-index.md#nested-tmux) | `Sync(run_tmux_command)` | `core_impl/tmux_dispatch.rs` `DISPATCH_303` | **RFC0002-TMUX** |
| `token` | [`token`](dispatcher-route-ledger-index.md#nested-token) | `Sync(token_run_command)` | `core_impl/token.rs` `DISPATCH_106` | **HOST-ADMIN** |
| `tokens` | — | `Sync(token_tokens_alias_command)` | `core_impl/token.rs` `DISPATCH_106` | **HOST-ADMIN** |
| `tonk` | [`tonk`](dispatcher-route-ledger-index.md#nested-tonk) | `Sync(tonk_run_command)` | `core_impl/tonk_oracle.rs` `DISPATCH_119` | **COMPAT/UNRESOLVED** |
| `transport` | [`transport`](dispatcher-route-ledger-index.md#nested-transport) | `Sync(run_transport_plan)` | `core_impl/transport_policy.rs` `DISPATCH_323` | **HOST-ADMIN** |
| `triggers` / `trigger` | — | `Sync(run_triggers_command_136)` | `core_impl/triggers.rs` `DISPATCH_144` | **COMPAT/UNRESOLVED** |
| `trust` / `trusts` | [`trust`](dispatcher-route-ledger-index.md#nested-trust) | `Sync(trust_run_command)` | `core_impl/trust.rs` `DISPATCH_98` | **HOST-ADMIN** |
| `ui` | — | `Sync(ui_run_command)` | `core_impl/maw_ui.rs` `DISPATCH_100` | **COMPAT/UNRESOLVED** |
| `update` | — | `Sync(update_run_command)` | `core_impl/update.rs` `DISPATCH_150` | **COMPAT/UNRESOLVED** |
| `upgrade` | — | `Sync(upgrade_run_command)` | `core_impl/update.rs` `DISPATCH_150` | **COMPAT/UNRESOLVED** |
| `user-setup` | [`user-setup`](dispatcher-route-ledger-index.md#nested-user-setup) | `Sync(run_usersetup_command)` | `core_impl/user_setup.rs` `DISPATCH_68` | **COMPAT/UNRESOLVED** |
| `--version` / `-v` / `version` | — | `Sync(version_handler)` | `core_impl/dispatcher.rs` `DISPATCH_01` | **HOST-ADMIN** |
| `view` | — | `Sync(view_run_command)` | `core_impl/tmux_readonly_view.rs` `DISPATCH_260` | **RFC0002-TMUX** |
| `wake` | — | `Async(wake_async_native)` | `core_impl/wake.rs` `DISPATCH_64` | **KERNEL5** |
| `wave` | [`wave`](dispatcher-route-ledger-index.md#nested-wave) | `Sync(wave_run_command)` | `core_impl/wave.rs` `DISPATCH_326` | **EXTERNAL-963** |
| `whoami` | — | `Sync(run_whoami_command)` | `core_impl/whoami.rs` `DISPATCH_36` | **COMPAT/UNRESOLVED** |
| `work` | — | `Sync(work_run_command)` | `core_impl/workspace_scaffold_commands.rs` `DISPATCH_93` | **KERNEL5** |
| `workon` | — | `Sync(run_workon_command)` | `core_impl/workon.rs` `DISPATCH_49` | **COMPAT/UNRESOLVED** |
| `workspace` / `ws` | [`workspace`](dispatcher-route-ledger-index.md#nested-workspace) | `Sync(run_workspace_command)` | `core_impl/workspace.rs` `DISPATCH_140` | **COMPAT/UNRESOLVED** |
| `worktree` | [`worktree`](dispatcher-route-ledger-index.md#nested-worktree) | `Sync(run_worktree_command)` | `core_impl/worktree.rs` `DISPATCH_145` | **COMPAT/UNRESOLVED** |
| `worktree-window` | — | `Sync(run_worktree_window_plan)` | `core_impl/worktree_window.rs` `DISPATCH_310` | **COMPAT/UNRESOLVED** |
| `x` | [`x`](dispatcher-route-ledger-index.md#nested-x) | `Sync(run_x_command)` | `core_impl/x.rs` `DISPATCH_335` | **HOST-ADMIN** |
| `xdg` | [`xdg`](dispatcher-route-ledger-index.md#nested-xdg) | `Sync(run_xdg_plan)` | `core_impl/hub_xdg_plan.rs` `DISPATCH_300` | **COMPAT/UNRESOLVED** |
| `zai` | [`zai`](dispatcher-route-ledger-index.md#nested-zai) | `Async(zai_run_async)` | `core_impl/zai.rs` `DISPATCH_327` | **COMPAT/UNRESOLVED** |
| `zoom` | — | `Sync(zoom_run_command)` | `core_impl/tmux_zoom.rs` `DISPATCH_80` | **RFC0002-TMUX** |

Rows: **91**; source spellings covered: **106**.
