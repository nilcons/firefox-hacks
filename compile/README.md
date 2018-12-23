# Official documentation

The official documentation on building Firefox is kind of OK, so make
sure you are familiar with that.  Compared to those, I will only
provide a couple of tips.

- https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions
- https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/Simple_Firefox_build
- https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/Configuring_Build_Options

# Mercurial vs Git and repository locations

The repo is of course huge, so it's important to realize that the
repository addresses contained in the instructions are the
bleeding-edge repositories.

They do not contain the proper commits and tags to hack on official
released versions of Firefox.

So before you clone 5GBs of repositories, make sure that if you want
to just change the behavior of the latest stable version a little bit
for your personal usage, then you use the proper repository:
https://hg.mozilla.org/releases/mozilla-release/

If you prefer to use git (which has a much-much faster `git grep` than
`hg grep`, and also has `git clean`), then you can use this mirror:
https://github.com/mozilla/gecko-dev/tree/release

Unfortunately, the git mirror doesn't have the release tags, but the
release branch is pointing to the latest released desktop version and
that is the one you usually want to edit anyways.

# Installing the build dependencies

Just use `apt build-dep firefox` or follow the official documentation,
I didn't have any issues.

# Coming up with a build configuration file

Use this as your `mozconfig` (in the root directory of the git repository):

```
mk_add_options MOZ_MAKE_FLAGS=-j6
ac_add_options --with-ccache=/usr/bin/ccache

ac_add_options --enable-optimize
ac_add_options --disable-debug
ac_add_options --disable-tests
ac_add_options --enable-strip
# ac_add_options --enable-profile-guided-optimization

export MOZILLA_OFFICIAL=1
mk_add_options MOZ_CO_PROJECT=browser
ac_add_options --enable-application=browser
ac_add_options --enable-official-branding
ac_add_options --enable-startup-notification

ac_add_options --prefix=/opt/firefox
ac_add_options --with-user-appdir=.gregzilla

ac_add_options --disable-gconf
ac_add_options --disable-negotiateauth
ac_add_options --disable-dbus
ac_add_options --disable-necko-wifi
ac_add_options --disable-updater
ac_add_options --disable-accessibility
ac_add_options --disable-crashreporter
ac_add_options --disable-parental-controls
# 2018-12-21: For some reason startupcache disabling doesn't work
# ac_add_options --disable-zipwriter
# ac_add_options --disable-startupcache

# Useful during development, for faster build times
# ac_add_options --with-system-nss
# ac_add_options --with-system-jpeg
# # ac_add_options --with-system-png # debian libpng doesn't have apng support
# ac_add_options --with-system-zlib
# ac_add_options --with-system-bz2
# ac_add_options --with-system-nspr
# ac_add_options --with-system-libevent
# ac_add_options --with-system-libvpx
# ac_add_options --with-system-icu
# ac_add_options --enable-system-sqlite
# ac_add_options --enable-system-pixman
```

This is according to my taste, feel free to change as you wish.  Using
`ccache` is a big help when you change configurations.  Without it, a
clean build is 50-60 minutes on my machine, while with it it's only
10-15.  You have to increase the default cache size, so it can
accomodate Firefox, I used 15G and that was more than enough.

Rust compilation is super slow, so using `sccache` instead of `ccache`
might be beneficial, but I haven't tried to set it up.

If you do a `git clean -dfx` during development to get a clean
recompile, don't forget to put back the `mozconfig` file after!

# Take care of your cgroups

If you don't have proper resource isolation set up, the build can
severly slow down your desktop.  If you need help with this, I have
another post about cgroups: https://github.com/nilcons/cgroup-infos

# Building

Use `./mach build`, no need to configure or anything like that.

Also, the build system seems OK regarding incremental builds: if you
only change a source code file (be it JavaScript or C++), not the
configuration and you do a `./mach build`, then it finished for me in
1-2 minute at most.  I haven't found a case where it failed to rebuild
correctly after a source change.

Once the build is finished, the result you can find in
`obj-x86_64-pc-linux-gnu/dist/`, so you can run it with
`obj-x86_64-pc-linux-gnu/dist/bin/firefox`.

# My patches

`0001-Move-.mozilla-.gregzilla.patch`: changes `~/.mozilla` to
`~/.gregzilla`, if you are not using this patch, then you have to
remove `ac_add_options --with-user-appdir=.gregzilla` from the
mozconfig (see https://bugzilla.mozilla.org/show_bug.cgi?id=298784).

0002-Remap-ctrl-t-to-BrowserHome-button-1.patch: the source code
version of remapping Ctrl-T to home middle click.

0003-Add-ctrl-alt-p-n-shortcuts-for-previous-next-tab.patch: the
source code version of adding Ctrl+Alt+{p,n} for previous and next
tab.

0004-Mozconfig-for-errge.patch: my mozconfig as a patch.

0005-dev-shm-org.mozilla.ipc-dev-shm-org.gregzilla.ipc.patch: changing
`/dev/shm/org.mozilla.ipc` to `/dev/shm/org.gregzilla.ipc`.  Not
really useful or important, just wanted to strace my firefox and make
sure that no `.mozilla` file access is ever used and this was in my
way :)
