# State of the web browsers (XMas of 2018)

So, naturally, I start with a rant.

BROWSERS ARE STILL SO BAD FOR POWERUSERS, UNBELIEVABLE!

Of course, my friends will say I'm a perfectionist. NO, I SHOULD GET
NEW FRIENDS!

Chrome and Firefox: am I a perfectionist, if I want to change my
keyboard bindings?  Yes, even in tabs which are
super-secure-non-extension-hackable-tabs (so I can roll through my
tabs by holding the next-tab shortcut without it stopping on a
"secure" tab)?  I don't want to go back to the previous tab with
Ctrl+Shift+Tab!  How many fingers and arms do these developers have?
Are they human or some other species?

Chrome and Firefox: am I a perfectionist if I want to set my own new
tab page to a `file://` based checkout of Hoplax
(https://github.com/hoplax/hoplax)?  Yes, the new tab page is on my
own computer, it's not your adspace to use as you wish.  Mozilla
putting the user first?  Last time in 1998, definitely not in 2019.
(Also: once that page is loaded, focus should be on the page, not in
the address bar.  Sad, that I have to say this explicitly.)

Chrome: am I a perfectionist if I want to have my tabs on the left
(not at the top)?  Come on, are you guys still developing on 4:3
screens in the Google offices?  Are you all using your displays in a
standing position?  News for you: your users are not!  Also: vertical
space is expensive!  You redesign GMail every week, but can't ask a UI
designer to tell you this triviality?

Chrome and Firefox: once you make it possible to have vertical tabs,
also please make it possible to hide the original normal tabs?  Maybe
I have to spell it out for you: OTHERWISE THE VERTICAL TABS ARE NOT
USEFUL!  (To be fair, in Firefox at least you can hack this with
`chrome/userChrome.css`.)

Chrome, Vivaldi: am I a perfectionist if I don't plan to use 15
different `--user-data-dir` profiles just to have some privacy and
security?  Also, I don't want to have a different window open for
every profile.  Make Firefox like container tabs a reality on the
Chromium engine please!

I'll tell you a couple of more requirements, that I gave up on, just
so you see that I'm not a perfectionist (all of this worked perfectly
before the Mozilla team decided to git clone chromium after Firefox 57):
  - video download helper without external programs,
  - tamper data that just works.

Other requirements, that are not part of the rant, just here for
future reference, so I know what to check in a browser:
  - does it have a hit-a-hint like extension (like Surfingkeys)?
  - does it support Flash via PPAPI (NPAPI is not maintained by adobe)?
  - does it have a referer control extension?
  - does it have a user agent switcher extension?
  - does it have a sixornot like extension?
  - does it have good adblock?

# Quick evaluation: Chrome, Vivaldi, Firefox (Quantum, not pre-57)

Chrome:
  - no clear way to vertical tabs at all, not with any amount of
    hacking or extensions.  Funny story, they had this feature for
    Windows and OSX, but the GNU/Linux guys complained so loudly, that
    Google simply removed the feature from the Windows and OSX version
    of Chrome.  I wonder if they will be cancelling on Google Drive if
    the community asks for a GNU/Linux client too loudly?
  - no way to remap keys in a way that they always work,
  - can't change new tab page to Hoplax,
  - also no plans for containers,
  - open source (?).

Vivaldi:
  - it has everything out of the box and also kind of nice,
  - a little bit slow though,
  - and it doesn't have containers,
  - closed source.

Firefox:
  - has vertical tabs (have to hack `chrome/userChrome.css`),
  - can't change new tab page to Hoplax (`file://`),
  - no way to remap keys in a way that they always work,
  - open source.

As you can see, the closest match is either Vivaldi or Firefox, but
Vivaldi is closed source, so I will be hacking Firefox today!

# Compiling and hacking the Firefox source code

See the `compile/` subdirectory.  After having this, I found out that
I can achieve everything by just changing things in a downloaded
binary release, so I no longer compile my own Firefox, the info is
just kept for historical reasons.  And for the case if they screw up
so much in the future that I need to compile it again.

# Binary hacking Firefox

To have all my requirements from the rant fulfilled, strictly speaking
I only need two small changes (the rest can be achieved with extensions):

  - Ctrl+Alt+N: next tab,
  - Ctrl+Alt+P: previous tab,
  - Ctrl+T: open new tab at a URL (`file://` based Hoplax in my case).

The first two are just simple keyboard bindings that I will explain
how to do without recompilation and without addons soon.

The third one is the tricky one, that makes it even trickier is that
I need the new page focused instead of the URL bar after loading.

Fortunately, I have noticed the following: you can still change the
home page to your custom URL in Firefox.  Then you will have this home
page in every new window (but not new tab) and also when you click the
home icon.  Also, if you middle click the home icon, then this home
page opens in a new tab (and it is focused, not the URL bar).

Therefore, I will implement Ctrl+T by simply unbinding the current new
tab command and rebiding it to the same command that the middle mouse
button executes on the home icon.

The advantage of binary hacking compared to recompilation (apart from
being faster), is that I can keep the Debian package and just run my
automated hacks after apt-get updated the package.  So I get quite
normal security updates and I just run my scripts if I notice that my
Firefox bindings are broken again.

We haven't fulfilled the requirement that keybindings from extensions
should work on the chrome pages (like server not found), but having
Ctrl+Alt+{N,P} is enough for me, as this makes it possible to at least
get away from those pages without closing them and without using the
mouse or some inconvenient other keyboard shortcut.

## Extracting the relevant binary file

```shell
cd /usr/lib/firefox/browser
mv omni.ja omni.ja.orig
mkdir x; cd x
unzip ../omni.ja    # ignore the warnings, it's fine!
```

## Keyboard shortcuts

The relevant keyboard shortcuts are stored in
`./chrome/browser/content/browser/browser.xul`.

This is the only file I need to patch:
```patch
--- /usr/lib/firefox/browser/x/chrome/browser/content/browser/browser.xul	2010-01-01 00:00:00.000000000 +0100
+++ ./chrome/browser/content/browser/browser.xul	2018-12-23 16:17:02.344249674 +0100
@@ -282,8 +282,9 @@
          key="&newNavigatorCmd.key;"
          command="cmd_newNavigator"
          modifiers="accel" reserved="true"/>
-    <key id="key_newNavigatorTab" key="&tabCmd.commandkey;" modifiers="accel"
-         command="cmd_newNavigatorTabNoEvent" reserved="true"/>
+    <key id="key_newNavigatorTab" key="&tabCmd.commandkey;" modifiers="accel" oncommand="BrowserHome({ button: 1 });" reserved="true"/>
+    <key id="gregzilla_prevTab" key="p" modifiers="accel,alt" oncommand="gBrowser.tabContainer.advanceSelectedTab(-1, true);" reserved="true"/>
+    <key id="gregzilla_prevTab" key="n" modifiers="accel,alt" oncommand="gBrowser.tabContainer.advanceSelectedTab(1, true);" reserved="true"/>
     <key id="focusURLBar" key="&openCmd.commandkey;" command="Browser:OpenLocation"
          modifiers="accel"/>
     <key id="focusURLBar2" key="&urlbar.accesskey;" command="Browser:OpenLocation"
```

We are just doing the 3 minimal modifications we have talked about:
  - remapping ctrl-t to be equivalent to middle clicking the home button,
  - adding ctrl+alt+n for next tab,
  - adding ctrl+alt+p for previous tab.

If this all works out, I may add more shortcuts this way, much more
resilient than remapping via webextensions.

## Pack up the `omni.ja` again

```shell
zip -qr9XD ../omni.ja *
```

## Deleting the startup cache from your profile

In your home directory, you have to delete the Firefox `startupCache`
directories (every profile has one, but not in the profile dir).  This
is necessary, so Firefox rereads the `omni.ja` file and our keyboard
shortcuts start to work.

Here is a one-liner:

```shell
find ~/.cache/mozilla/firefox -type d -name startupCache | xargs rm -rf
```

## Start it up

It's done, enjoy your new shortcuts!  Don't forget to
clone [Hoplax](http://github.com/hoplax/hoplax), set it up as your
homepage in Firefox preferences and use it as your new tab page!

# Binary hacking automated

Also called `patch-the-fox`:
```shell
#!/bin/bash

set -e

tempdir=$(mktemp -d)
mkdir "$tempdir/extract"
cd "$tempdir/extract"
set +e
unzip /usr/lib/firefox/browser/omni.ja
if [ "$?" -ne 2 ]; then
  echo >&2 "Unexpected exit code from unzip"
  exit 1
fi
set -e
patch -p1 <<EOF
--- /usr/lib/firefox/browser/x/chrome/browser/content/browser/browser.xul	2010-01-01 00:00:00.000000000 +0100
+++ ./chrome/browser/content/browser/browser.xul	2018-12-23 16:17:02.344249674 +0100
@@ -282,8 +282,9 @@
          key="&newNavigatorCmd.key;"
          command="cmd_newNavigator"
          modifiers="accel" reserved="true"/>
-    <key id="key_newNavigatorTab" key="&tabCmd.commandkey;" modifiers="accel"
-         command="cmd_newNavigatorTabNoEvent" reserved="true"/>
+    <key id="key_newNavigatorTab" key="&tabCmd.commandkey;" modifiers="accel" oncommand="BrowserHome({ button: 1 });" reserved="true"/>
+    <key id="gregzilla_prevTab" key="p" modifiers="accel,alt" oncommand="gBrowser.tabContainer.advanceSelectedTab(-1, true);" reserved="true"/>
+    <key id="gregzilla_prevTab" key="n" modifiers="accel,alt" oncommand="gBrowser.tabContainer.advanceSelectedTab(1, true);" reserved="true"/>
     <key id="focusURLBar" key="&openCmd.commandkey;" command="Browser:OpenLocation"
          modifiers="accel"/>
     <key id="focusURLBar2" key="&urlbar.accesskey;" command="Browser:OpenLocation"
EOF
zip -qr9XD ../omni.ja *
sudo bash -c "cp /usr/lib/firefox/browser/omni.ja /usr/lib/firefox/browser/omni.ja.orig ; cat $tempdir/omni.ja >/usr/lib/firefox/browser/omni.ja"
find ~/.cache/mozilla/firefox -type d -name startupCache | xargs rm -rf
cd /
rm -r "$tempdir"
```

# Making Hoplax real fast and input focus issues

If you use Hoplax as your new tab page with `file://`, it may help if
you tab pin one instance of it and just leave it alone as your first
tab or something.  As long as you have one Hoplax tab loaded, Firefox
will load the new instances much-much quicker.

If you are using Surfingkeys, then use Alt+S to turn off it on Hoplax,
or if you don't prefer this solution, then just use the I button after
a new tab is loaded to focus the Hoplax main inputbox.

# Getting rid of the original tabbar (if you are using vertical tabs)

If you are using the "Vertical Tabs Reloaded" (or similar) extension,
then you want to do the famous `chrome/userChrome.css` hack to hide
the original tabbar.

Go to your profile dir (`~/.mozilla/firefox/<your-profile-dir>`):

```shell
mkdir -p chrome
cat <<EOF >chrome/userChrome.css
@namespace url(http://www.mozilla.org/keymaster/gatekeeper/there.is.only.xul);

/* This disables the tabbar */
#TabsToolbar {
  visibility:collapse!important;
}

/* This disables sidebar headers (history sidebar, tab sidebar, etc.) */
/* Since every vertical tab addon we will try have a different id, we just disable all sidebar headers */
#sidebar-box #sidebar-header {
  display: none;
}
EOF
```

# Firefox safety and privacy

You can use https://ffprofile.com/ to create a default profile which
is a little bit more private and safe than the default.

After going through the settings, my `user.js` looks like this:
```js
user_pref("app.normandy.api_url", "");
user_pref("app.normandy.enabled", false);
user_pref("app.shield.optoutstudies.enabled", false);
user_pref("app.update.auto", false);
user_pref("breakpad.reportURL", "");
user_pref("browser.crashReports.unsubmittedCheck.autoSubmit", false);
user_pref("browser.crashReports.unsubmittedCheck.autoSubmit2", false);
user_pref("browser.crashReports.unsubmittedCheck.enabled", false);
user_pref("browser.disableResetPrompt", true);
user_pref("browser.newtab.preload", false);
user_pref("browser.newtabpage.activity-stream.section.highlights.includePocket", false);
user_pref("browser.newtabpage.enabled", false);
user_pref("browser.newtabpage.enhanced", false);
user_pref("browser.newtabpage.introShown", true);
user_pref("browser.safebrowsing.appRepURL", "");
user_pref("browser.safebrowsing.blockedURIs.enabled", false);
user_pref("browser.safebrowsing.downloads.enabled", false);
user_pref("browser.safebrowsing.downloads.remote.enabled", false);
user_pref("browser.safebrowsing.downloads.remote.url", "");
user_pref("browser.safebrowsing.enabled", false);
user_pref("browser.safebrowsing.malware.enabled", false);
user_pref("browser.safebrowsing.phishing.enabled", false);
user_pref("browser.selfsupport.url", "");
user_pref("browser.shell.checkDefaultBrowser", false);
user_pref("browser.startup.homepage_override.mstone", "ignore");
user_pref("browser.tabs.crashReporting.sendReport", false);
user_pref("browser.urlbar.speculativeConnect.enabled", false);
user_pref("browser.urlbar.trimURLs", false);
user_pref("datareporting.healthreport.service.enabled", false);
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("datareporting.policy.dataSubmissionEnabled", false);
user_pref("dom.battery.enabled", false);
user_pref("dom.event.clipboardevents.enabled", false);
user_pref("experiments.activeExperiment", false);
user_pref("experiments.enabled", false);
user_pref("experiments.manifest.uri", "");
user_pref("experiments.supported", false);
user_pref("extensions.autoDisableScopes", 14);
user_pref("extensions.getAddons.cache.enabled", false);
user_pref("extensions.getAddons.showPane", false);
user_pref("extensions.greasemonkey.stats.optedin", false);
user_pref("extensions.greasemonkey.stats.url", "");
user_pref("extensions.pocket.enabled", false);
user_pref("extensions.shield-recipe-client.api_url", "");
user_pref("extensions.shield-recipe-client.enabled", false);
user_pref("extensions.webservice.discoverURL", "");
user_pref("general.warnOnAboutConfig", false);
user_pref("media.autoplay.default", 0);
user_pref("media.autoplay.enabled", true);
user_pref("network.IDN_show_punycode", true);
user_pref("network.allow-experiments", false);
user_pref("network.captive-portal-service.enabled", false);
user_pref("network.dns.disablePrefetch", true);
user_pref("network.http.speculative-parallel-limit", "0");
user_pref("network.prefetch-next", false);
user_pref("network.trr.mode", 5);
user_pref("privacy.donottrackheader.enabled", true);
user_pref("privacy.donottrackheader.value", 1);
user_pref("privacy.trackingprotection.enabled", true);
user_pref("privacy.trackingprotection.pbmode.enabled", true);
user_pref("privacy.usercontext.about_newtab_segregation.enabled", true);
user_pref("toolkit.telemetry.archive.enabled", false);
user_pref("toolkit.telemetry.bhrPing.enabled", false);
user_pref("toolkit.telemetry.cachedClientID", "");
user_pref("toolkit.telemetry.enabled", false);
user_pref("toolkit.telemetry.firstShutdownPing.enabled", false);
user_pref("toolkit.telemetry.hybridContent.enabled", false);
user_pref("toolkit.telemetry.newProfilePing.enabled", false);
user_pref("toolkit.telemetry.prompted", 2);
user_pref("toolkit.telemetry.rejected", true);
user_pref("toolkit.telemetry.reportingpolicy.firstRun", false);
user_pref("toolkit.telemetry.server", "");
user_pref("toolkit.telemetry.shutdownPingSender.enabled", false);
user_pref("toolkit.telemetry.unified", false);
user_pref("toolkit.telemetry.unifiedIsOptIn", false);
user_pref("toolkit.telemetry.updatePing.enabled", false);
```

Other projects with similar goals:
  - https://github.com/pyllyukko/user.js
  - https://github.com/intika/Librefox-Firefox

# Version information

Binary hacking has been last checked for Firefox 64.0, on 2018-12-23.

Source compilation instructions has been checked for Firefox 64.0, on 2018-12-23.

Both checks were done on an up-to-date Debian GNU/Linux sid amd64
machine.
