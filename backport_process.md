# [v7,0/5] drm/i915: Add support for Intel's eDP backlight controls” backport to `Ubuntu-oem-5.10-5.10.0-1020.21`

I will describe here the process I followed for this task:

*This is a markdown format file, it is recommended to use a markdown view tool
for full the "experience"*.

1. I cloned the repository:

> git clone https://git.launchpad.net/~canonical-kernel/ubuntu/+source/linux-oem/+git/focal

2. I then checkout to release tag `Ubuntu-oem-5.10-5.10.0-1020.21`.

> git checkout Ubuntu-oem-5.10-5.10.0-1020.21

3. I created my own branch based in the release tag:

> git checkout -b ikersustatxa/backport_Intel_eDP_backlight_controls

4. I created a repository in github where I could push my changes, added the
repo address to the "git workspace" and pushed my branch there:

> git remote add my_repo git@github.com:ikerperezdelpalomar/ubuntu_backport.git

> git push my_repo

5. I read the patchset description, and I had a look at the changes in first
commit:

> git show 31b10c1a947d

6. I tried to apply the commit:

> git cherry-pick 31b10c1a947d

There were conflicts in the next files:

```
both modified:   drivers/gpu/drm/i915/display/intel_display_types.h
both modified:   drivers/gpu/drm/i915/display/intel_dp_aux_backlight.c
both modified:   drivers/gpu/drm/i915/display/intel_panel.c
```

7. I decided to have a look to the history of each file, to check which
changes have been trough before the `31b10c1a947d` commit. I decided to start
with `intel_display_types.h`, I checked the commits that affected that file in
`oem` branch history:

> git log oem -- drivers/gpu/drm/i915/display/intel_display_types.h

But the command didn't output anything, it seemed like the file wasn't located
in the same directory anymore, I double checked:

> vim drivers/gpu/drm/i915/display/intel_display_types.h

And I confirmed the file was not there anymore.

8. I decided to checkout to the point in history corresponding to commit
`31b10c1a947d` and create a branch from it:

> git checkout oem

> git checkout 31b10c1a947d

> git checkout -b iker/first_commit

9. I wanted to know which changes were done to the file from 
`Ubuntu-oem-5.10-5.10.0-1020.21` release to `31b10c1a947d`. To do so, I looked
for the commit history of the file in both branches:

> git log --oneline -- drivers/gpu/drm/i915/display/intel_display_types.h

<details>
    <summary>iker/first_commit branch. Click to expand</summary>

```
31b10c1a947d (HEAD -> iker/first_commit) drm/i915: Pass port to intel_panel_bl_funcs.get()
7853b437391a drm/i915/display: fix the uint*_t types that have crept in
e9fd05c3e4f2 drm/i915/hdcp: Support for HDCP 2.2 MST shim callbacks
5bd29e32bb99 drm/i915/hdcp: Pass connector to check_2_2_link
e03187e12cae drm/i915/hdcp: MST streams support in hdcp port_data
a6c6eac947d5 drm/i915/hdcp: Encapsulate hdcp_port_data to dig_port
1a67a168f57b drm/i915/hdcp: HDCP stream encryption support
fc6097d4fb29 drm/i915/hdcp: DP MST transcoder for link and stream
2bbd6dba84d4 drm/i915: Try to use fast+narrow link on eDP again and fall back to the old max strategy on failure
ca765c731ebd Merge tag 'drm-intel-next-2021-01-04' of git://anongit.freedesktop.org/drm/drm-intel into drm-next
b3304591f14b drm/i915/dp: Track pm_qos per connector
```

</details>


> git log ikersustatxa/backport_Intel_eDP_backlight_controls \
--oneline -- drivers/gpu/drm/i915/display/intel_display_types.h

<details>
    <summary> ikersustatxa/backport_Intel_eDP_backlight_controls branch. Click to expand </summary>

```
ba62ceec5ab3 UBUNTU: SAUCE: drm/i915: Switch to LTTPR non-transparent mode link training
a5169f2e71f5 UBUNTU: SAUCE: drm/i915: Switch to LTTPR transparent mode link training
214cb23e62a5 UBUNTU: SAUCE: drm/i915: Plumb crtc_state to link training
3fa12585afc2 drm/i915/dp: Track pm_qos per connector
```

</details>

This way, I found which were commits that were applied to both branches, from a
shared starting point commit (`3fa12585afc2 drm/i915/dp: Track pm_qos per connector`).

My plan was to decide which commits should I cherry-pick from `iker/first_commit`
to `ikersustatxa/backport_Intel_eDP_backlight_controls`, based on the changes
that were missing in each one of the files that before had a merge conflict.
However, I considered that this process was not appropriate. Because, this
would modify the git commit order in the history of the driver, compared to the
oem main branch. So, I needed to search fot the specific commits that affected 
to the driver, and find out the order in which I should pick them. 


> git log --oneline --grep="i915"
<details>
    <summary>ikersustatxa/backport_Intel_eDP_backlight_controls branch. Click to expand</summary>

```
c4ed3f685891 drm/i915: Wedge the GPU if command parser setup fails
48e013755c89 drm/i915: Reject 446-480MHz HDMI clock on GLK
8800f89cd771 drm/i915/gt: Correct surface base address for renderclear
f3ad521d8202 drm/i915/gt: Flush before changing register state
f7b8197e7d01 drm/i915/gt: One more flush for Baytrail clear residuals
64e81dfdba8a UBUNTU: SAUCE: drm/i915: Drop require_force_probe from JSL
ade9f1d9f14e drm/i915/dp: Program source OUI on eDP panels
a6facbbfd94e drm/i915: Init lspcon after HPD in intel_dp_detect()
59cef73c6ef3 drm/i915/tgl+: Make sure TypeC FIA is powered up when initializing it
7e38a33471b5 drm/i915: Fix overlay frontbuffer tracking
ee4fc760f06a drm/i915: Skip vswing programming for TBT
cd588e201cb7 drm/i915: Fix ICL MG PHY vswing handling
3207ce0d2cd2 drm/i915: Power up combo PHY lanes for for HDMI as well
aa19b900ad66 drm/i915: Extract intel_ddi_power_up_lanes()
f389aa656277 drm/i915/display: Prevent double YUV range correction on HDR planes
d9428dbc3e29 drm/i915/gt: Close race between enable_breadcrumbs and cancel_breadcrumbs
e9de36fb0a40 drm/i915/gem: Drop lru bumping on display unpinning
372fb2f0c8a4 stmmac: intel: Configure EHL PSE0 GbE and PSE1 GbE to 32 bits DMA addressing
2fa0cbee24ba drm/i915/selftest: Fix potential memory leak
e9046777b6e6 drm/i915: Check for all subplatform bits
39e98082e2f1 drm/i915/pmu: Don't grab wakeref when enabling events
433c6465b7b9 drm/i915/gt: Clear CACHE_MODE prior to clearing residuals
f253ad19cf6e drm/i915/gt: Always try to reserve GGTT address 0x0
c282877d9c7b drm/i915: Always flush the active worker before returning from the wait
8b246cdc63f3 UBUNTU: SAUCE: drm/i915/gen9bc: Handle TGP PCH during suspend/resume
f94762c8700f drm/i915: Update gen12 multicast register ranges
b4de13b409d1 drm/i915: Update gen12 forcewake table
c0c0ccdde2f6 drm/i915: Rename FORCEWAKE_BLITTER to FORCEWAKE_GT
b1385e901a7a drm/i915/rkl: Add new cdclk table
f025b399afc3 drm/i915/display/fbc: Implement WA 22010751166
13fcebbf8084 drm/i915/tgl: Fix typo during output setup
29eef1514c5d UBUNTU: SAUCE: drm/i915: Fix the MST PBN divider calculation
510ff4f70866 UBUNTU: SAUCE: drm/i915: Drop require_force_probe from RKL
270f96d07c84 drm/i915/hdcp: Get conn while content_type changed
71bb92a7284c ASoC: SOF: Intel: fix page fault at probe if i915 init fails
ff229361e6c7 drm/i915/hdcp: Update CP property in update_pipe
f96ee7557abd drm/i915: Only enable DFP 4:4:4->4:2:0 conversion when outputting YCbCr 4:4:4
4ed156bbef33 drm/i915: s/intel_dp_sink_dpms/intel_dp_set_power/
439ddbb199d0 drm/i915: Check for rq->hwsp validity after acquiring RCU lock
ad503653cf67 drm/i915/gt: Prevent use of engine->wa_ctx after error
427750536636 drm/i915/vbt: Add VRR VBT toggle
4de9a98d0ac2 drm/i915/vbt: Update the version and expected size of BDB_GENERAL_DEFINITIONS map
1f050cb3708e drm/i915/vbt: Fix backlight parsing for VBT 234+
d81227169196 UBUNTU: SAUCE: drm/i915/gen9_bc : Add TGP PCH support
b78f6e4d9577 drm/i915/rkl: new rkl ddc map for different PCH
2507ac07d98d drm/i915: s/PORT_TC/TC_PORT_/
431d9ff01ca3 drm/i915: Add PORT_TCn aliases to enum port
d4bc0772f611 drm/i915/jsl: Split EHL/JSL platform info and PCI ids
b3532e725eab drm/i915/display/ehl: Limit eDP to HBR2
3e9d9ecdeced drm/i915/dg1: add hpd interrupt handling
610ec934549f drm/i915/dg1: Don't program PHY_MISC for PHY-C and PHY-D
85a2bdaea24d drm/i915/dg1: gmbus pin mapping
...skipping...
3fa12585afc2 drm/i915/dp: Track pm_qos per connector
```

</details>


>  git log iker/first_commit --oneline --grep="i915"

<details>
    <summary> iker/first_commit branch. Click to expand</summary>

```
31b10c1a947d (iker/first_commit) drm/i915: Pass port to intel_panel_bl_funcs.get()
a1f6bfe17931 drm/i915: Try to guess PCH type even without ISA bridge
6b20b734bbf1 drm/i915/display: Bitwise or the conversion colour specifier together
ba8ef8c0b958 drm/i915: Drop one more useless master_transcoder assignment
35f0837e0682 drm/i915/dg1: Apply WA 1409120013 and 14011059788
d70920adf9f2 drm/i915/pps: rename intel_dp_init_panel_power_sequencer* functions
bcdf0f71b0e9 drm/i915/pps: rename vlv_init_panel_power_sequencer to vlv_pps_init
572a0d301754 drm/i915/pps: add locked intel_pps_wait_power_cycle
07eb5b1f1711 drm/i915/pps: rename intel_power_sequencer_reset to intel_pps_reset_all
c94287f158dc drm/i915/pps: rename intel_dp_check_edp to intel_pps_check_power_unlocked
73bb78b5ba68 drm/i915/pps: abstract intel_pps_encoder_reset()
c520869ac4ef drm/i915/pps: add higher level intel_pps_init() call
f033d7eb000a drm/i915/pps: abstract intel_pps_vdd_off_sync
db7c94f908ad drm/i915/pps: rename edp_panel_* to intel_pps_*_unlocked
eb46f498bf5f drm/i915/pps: rename intel_edp_panel_* to intel_pps_*
f4249942989b drm/i915/pps: rename intel_edp_backlight_* to intel_pps_backlight_*
7191d9d21b6f drm/i915/pps: rename pps_{,un}lock -> intel_pps_{,un}lock
abad6805ee78 drm/i915/pps: abstract panel power sequencer from intel_dp.c
7853b437391a drm/i915/display: fix the uint*_t types that have crept in
702c08d6d034 drm/i915/display: remove useless use of inline
67fba3f1c73b drm/i915/dp: Fix LTTPR vswing/pre-emp setting in non-transparent mode
1c6e527d6947 drm/i915/dp: Move intel_dp_set_signal_levels() to intel_dp_link_training.c
d5a0d4b9380a drm/i915/hdcp: Enable HDCP 2.2 MST support
899c8762f981 drm/i915/hdcp: Configure HDCP2.2 MST steram encryption status
e9fd05c3e4f2 drm/i915/hdcp: Support for HDCP 2.2 MST shim callbacks
d631b984cc90 drm/i915/hdcp: Add HDCP 2.2 stream register
5bd29e32bb99 drm/i915/hdcp: Pass connector to check_2_2_link
...skipping...
b3304591f14b drm/i915/dp: Track pm_qos per connector
```
</details>


This were too many commits, I had to reconsider my options. I decided to go back
to my previous plan and select the commits that affected the files in merge
conflict. When the conflicts were resolved, I could tidy up the git history
with an interactive git rebase.

Back to the previous situation I decided to search in `iker/first_commit`for
the two commits that were present in
`ikersustatxa/backport_Intel_eDP_backlight_controls` branch after
`b3304591f14b drm/i915/dp: Track pm_qos per connector` commit but they weren't
after `b3304591f14b` in `iker/first_commit`. These are:

```
ba62ceec5ab3 UBUNTU: SAUCE: drm/i915: Switch to LTTPR non-transparent mode link training
a5169f2e71f5 UBUNTU: SAUCE: drm/i915: Switch to LTTPR transparent mode link training
```

I found out that they existed in `iker/first_commit`, however there were located
way behind `b3304591f14b drm/i915/dp: Track pm_qos per connector` commit in git
history. That clearly clashed with my previous concerns about respecting git
history as completely sacred, and confirmed I could go for my second plan.

9. I decided to start cherry-picking commits one by one to check the effect they
would produce. So I tried with the `ca765c731ebd Merge tag 'drm-intel-next-2021-01-04' of git://anongit.freedesktop.org/drm/drm-intel into drm-next` commit:

> git cherry-pick -m 1 ca765c731ebd

This produced a lot of changes and a lot of conflicts, so I thought I might be
brutalizing the process a bit

10. In order to make sure I was solving the conflicts properly, I opened each of
the conflicting files at the point in history of `31b10c1a947 ddrm/i915: Pass`
`port to intel_panel_bl_funcs.get()` commit. This way, I could double-check
how the file should look at the end of the cherry-picking process.

> git show 31b10c1a947d:${DESIRED_FILE}

After solving the conflicts, and cherry-picking all the way up until the last
commit in the patch set, I wanted to double check that the rest of the files
that conflicted at the beginning of this task, looked in the same way that they
looked in the patchset.

11. I created a new branch based in my work branch and I rolled back to the first commit of the patchset:

> git checkout -b iker/double_check

> git reset --hard f48b5b1b9cc1cb44c02bd9d02fd82a025ea275dc

12. I compared the files between the two branches

> git diff iker/double_check..iker/first_commit -- ${FILES}

Being FILES = 

    - drivers/gpu/drm/i915/display/intel_display_types.h
    - drivers/gpu/drm/i915/display/intel_dp_aux_backlight.c
    - drivers/gpu/drm/i915/display/intel_panel.c


Even thought `intel_panel.c` file was exactly the same in both branches,
`intel_display_types.h` and `intel_dp_aux_backlight.c` showed many differences
between `iker/double_check`and `iker/first_commit` branches. I suppose this
happened because I missed some commits. I decided that `ikersustatxa/`
`backport_Intel_eDP_backlight_controls` would stay as it was and 
`iker/double_check` would be a parallel working branch where I would try to
sort this out. So keep in mind tha the work described below was done in
`iker/double_check`..

13. I decided to check which commits affected the code that was different in
the previous diff:

> git blame drivers/gpu/drm/i915/display/intel_display_types.h

First commit that I found was `07f9cd0b3870 drm/i915: make sure VDD is turned`
`off during system suspend`. This changed line 214 of file 
`drivers/gpu/drm/i915/display/intel_display_types.h`. The code shown in my branch
was different to the one in the `iker/first_commit` branch, which should be how
it should look like at the end of the task.

So I checked out into my branch, `git log` -ed and I searched for that commit, with the surprise that
I found it! It is there, so there must be a commit that later on changed
that code. I decided to step back from here and search for other changes.

Next commits I found that were missing are:

<details>
    <summary>Click to expand</summary>

```
a582354c92d10 drm/i915: Pimp the watermark documentation a bit
ced42f2df5fd8 drm/i915: Add support for starting FRL training for HDMI2.1 via PCON
c06e59b67e28 drm/i915: Add hw.pipe_mode to allow bigjoiner pipe/transcoder split
522508b665df3 rm/i915/display: Let PCON convert from RGB to YCbCr if it can
2f78347e36348 drm/i915: Capture max frl rate for PCON in dfp cap structure
ab01630b64ce1 drm/i915: Store plane relative data rate in crtc_state
0385ecead5178 drm/i915: HW state readout for Bigjoiner case
9c31212b24783 drm/i915: Precompute can_sagv for each wm level
6d1a2fdedb268 drm/i915: Enable scaling filter for plane and CRTC
19f65a3dbf75b drm/i915: Try to make bigjoiner work in atomic check
230edf78ed4b9 drm/i915: Add plane .{min,max}_width() and .max_height() vfuncs
b039193d12834 drm/i915: Add dedicated plane hook for async flip case
b9d96dacdc3d9 drm/i915: Read DSC capabilities of the HDMI2.1 PCON encoder
100fe4c01efff drm/i915: Add an encoder .shutdown() hook
98e497e203a58 drm/i915/dpcd_bl: uncheck PWM_PIN_CAP when detect eDP backlight capabilities
8a25c4be583d8 drm/i915/params: switch to device specific parameters
```

</details>

This commits didn't exist in `iker/double_check`, like it happened with
`07f9cd0b3870 drm/i915: make sure VDD ...`. Therefore I had something to start
working with.

In order to cherry-pick the missing commits in the correct order, I had to
search for them in the git history and determine their chronology. 

Here the chronoly:

<details>
    <summary>Click to expand</summary>

```
522508b665df3 rm/i915/display: Let PCON convert from RGB to YCbCr if it can
b9d96dacdc3d drm/i915: Read DSC capabilities of the HDMI2.1 PCON encoder
98e497e203a58 drm/i915/dpcd_bl: uncheck PWM_PIN_CAP when detect eDP backlight capabilities ****** NOT SURE IF THIS IS THE RIGHT SPOT
8a25c4be583d8 drm/i915/params: switch to device specific parameters ****** NOT SURE IF THIS IS THE RIGHT SPOT
ced42f2df5fd8 drm/i915: Add support for starting FRL training for HDMI2.1 via PCON
2f78347e36348 drm/i915: Capture max frl rate for PCON in dfp cap structure
0385ecead5178 drm/i915: HW state readout for Bigjoiner case
19f65a3dbf75b drm/i915: Try to make bigjoiner work in atomic check
ab01630b64ce1 drm/i915: Store plane relative data rate in crtc_state
9c31212b24783 drm/i915: Precompute can_sagv for each wm level
a582354c92d10 drm/i915: Pimp the watermark documentation a bit
c06e59b67e28 drm/i915: Add hw.pipe_mode to allow bigjoiner pipe/transcoder split
230edf78ed4b9 drm/i915: Add plane .{min,max}_width() and .max_height() vfuncs
6d1a2fdedb268 drm/i915: Enable scaling filter for plane and C
100fe4c01efff drm/i915: Add an encoder .shutdown() hook
b039193d12834 drm/i915: Add dedicated plane hook for async flip case
```

</details>

I fixed the order of the commits I picked before and I added the ones missing:

> git rebase -i HEAD~20


After picking the commits in the order described above the changes of the
`07f9cd0b3870 drm/i915: make sure VDD ...` commit disappeared.


After this process I added the patch commits again, however files were still
different, so I had to go through the same process again:

New missing commits:

```
68fd1faa92a20 drm/i915: Reuse the async_flip() hook for the async flip disable w/a
8693ee2e378da drm/i915: Add plane vfuncs to enable/disable flip_done interrupt
777e687a0c651  drm/i915: split fdi code out from intel_display.c
8cf41f316e649 drm/i915: refactor pll code out into intel_dpll.c
```

Finally this included all the code that I was missing in my branch, however
there still code that should be removed, but I don't know how to search for
commits that remove code, just the ones that include code.

At this stage, I would ask somebody for advice.

In my github repo you will find two branches:

- ikersustatxa/backport_Intel_eDP_backlight_controls
- iker/double_check

I believe that **iker/sustatxa** is in a closer state to how it should look the
backport at the end of the task, however I wanted to leave both branches so it
can be better understood the process described here and "just in case".