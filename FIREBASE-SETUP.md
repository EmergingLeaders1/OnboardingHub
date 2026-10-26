# Setting up sign-in for the Onboarding Hub

The hub uses Firebase so that staff sign in with Google and their progress is saved to their account, not just their browser. Until the steps below are done, the hub works as before: no sign-in, with progress saved only in the browser.

This setup has been done: the project is `el-onboarding-hub` (database in London, `europe-west2`) and its config is already in `index.html`. The steps are kept here for reference.

## 1. Create the Firebase project

1. Go to <https://console.firebase.google.com> and click **Create a project**.
2. Name it, for example `el-onboarding-hub`. You can turn Google Analytics off.

## 2. Turn on Google sign-in

1. In the left menu, open **Build → Authentication** and click **Get started**.
2. On the **Sign-in method** tab, choose **Google**, switch it on, pick a support email and click **Save**.

## 3. Create the database

1. Open **Build → Firestore Database** and click **Create database**.
2. Choose a location close to your staff (for example `europe-west2`, London). This can't be changed later.
3. Start in **production mode**.
4. Open the **Rules** tab, replace everything with the rules below and click **Publish**.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Each person can only read and write their own progress.
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

To allow only Emerging Leaders email addresses, use this line instead of the `allow` line above:

```
      allow read, write: if request.auth != null && request.auth.uid == userId
                         && request.auth.token.email.matches('.*@emerging-leaders[.]net$');
```

## 4. Connect the hub to the project

1. Click the gear icon next to **Project Overview**, then **Project settings**.
2. Under **Your apps**, click the web icon `</>`, name the app `Onboarding Hub` and click **Register app**. Leave Firebase Hosting unticked.
3. Firebase shows a `firebaseConfig` block. Copy four of its values into `index.html`. Search the file for `FIREBASE_CONFIG`:

```js
const FIREBASE_CONFIG = {
  apiKey: 'paste apiKey here',
  authDomain: 'paste authDomain here',
  projectId: 'paste projectId here',
  appId: 'paste appId here',
};
```

These values are meant to be public. They identify the project but don't grant access. The rules from step 3 decide who can read and write data.

## 5. Put the hub online

Sign-in doesn't work when `index.html` is opened from someone's computer (a `file:///` address). Opened that way, the hub skips sign-in and saves progress in that browser only.

The official hub is published with GitHub Pages from the `EmergingLeaders1/OnboardingHub` repository at **https://emergingleaders1.github.io/OnboardingHub/**. Pushing changes to the `main` branch updates it within a minute or two.

## 6. Approve the web address for sign-in

In Firebase, open **Authentication → Settings → Authorized domains**, click **Add domain** and add `emergingleaders1.github.io`. `localhost` is already there for testing. Note that this is the hub's web address, not the staff email domain.

## Seeing everyone's progress

In Firebase, open **Firestore Database → Data → users**. Each person has one entry showing their name, email, `progressPercent` and when they last made progress (`updatedAt`).

## How it behaves

- The first time someone opens the hub, they sign in with Google. Any progress they had already made in that browser is carried into their account.
- After that, progress saves automatically and follows them to any device.
- **Sign out** (top right) clears the progress from that browser, so a shared computer starts fresh for the next person.
