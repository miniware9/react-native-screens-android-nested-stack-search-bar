# react-native-screens: on Android, a nested native stack removes the header's search bar

On Android, a header given `headerSearchBarOptions` shows no search icon when
its screen renders a nested native stack, here one whose own headers are
hidden, so that the outer header is the only one. The same options on a
screen with no nested stack show the icon.

`ScreenStackFragment.updateToolbarMenu` clears the menu it is given, then adds
a search item only when its own screen's header has a search bar. Every
`ScreenStackFragment` that is not a form sheet calls it, from
`onCreateOptionsMenu` and `onPrepareOptionsMenu`, whatever its header, and the
menu is the activity's one options menu, the toolbar of the header that set
the action bar. A nested stack's fragments are children of the screen fragment
they are nested in, and AndroidX runs a fragment's options-menu callbacks
before its children's. So the outer screen adds the search item, and the
nested stack's screen, whose header has no search bar, clears it.

## Steps

1. Start an Android emulator, then run `npm install` and
   `npx expo run:android`. Android 17 asks whether the app may find nearby
   devices, which the development build needs to load from Metro: allow it.
2. The app opens on "Nested stack". Its header shows the title alone.
3. Press **Open a screen with no nested stack**. Its header, given the same
   options, has the search icon: press it and type, and the text shows under
   "Search text".
4. Go back. The "Nested stack" header has no search icon again, and neither
   has the screen that **Push a screen onto the nested stack** opens.

Expected: every header shows the search icon, and the text typed into it
shows under "Search text".

## Observed

Debug build as the steps say, expo 58.0.23, React Native 0.88.1,
react-native-screens 4.30.1, @react-navigation/native-stack 7.21.0, Android 17
emulator (Pixel 9a, x86_64), 2026-11-07: neither "Nested stack" nor the screen
pushed onto it had a search icon. "No nested stack" had one, and "zeus" typed
into it showed under "Search text". A release build
(`npx expo run:android --variant release`) behaved the same.

With `ScreenStackFragment` updating the menu only from the screen whose header
owns the activity's action bar, made in the installed
`node_modules/react-native-screens`, a release build showed the icon on all
three screens, and text typed into it on "Nested stack" showed under "Search
text".
