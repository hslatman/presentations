# REST in peace

---

# WHOAMI

<ul>
  <li>
    University of Twente
    <ul>
      <li>BSc. Technische Informatica</li>
      <li>MSC. Computer Science (Kerckhoff's Institute)</li>
    </ul>
  </li>
  <li>
    Software Architect at Extendas
    <ul>
        <li>Since ... 2016</li>
    	<li>Backend (PHP, Python)</li>
  	    <li>Mobile (Swift)</li>
    </ul>
  </li>
  <li>
    Contact information @ <a target="_blank" href="https://hermanslatman.nl/contact.html">https://hermanslatman.nl/contact.html</a>
  </li>
</ul>

---

# Introduction

<ul>
    <li>
        Mobile app development basics
    </li>
    <li>
        Frustrations in mobile app development
    </li>
    <li>
        Fast tracking app development with RMP
    </li>
    <li>
        Demo
    </li>
</ul>

---

# Components of a mobile app

<ul>
  <li>
    Business Logic
    <ul>
        <li>Models the domain of the application</li>
        <li>Defines the functionality of your app</li>
    </ul>
  </li>
  <li>
    Storage
    <ul>
    	<li>Remember stuff</li>
    	<li>Can basically be anything</li>
    </ul>
  </li>
  <li>
    Networking
    <ul>
        <li>Interact with web services</li>
        <li>Retrieve and send data</li>
        <li>An online backend</li>
    </ul>
  </li>
    <li>
      User Interface
      <ul>
        <li>Defines what users see</li>
        <li>Allows users to interact with the app</li>
      </ul>
    </li>
    <li>
        Hardware
    </li>
    <li>
        Diagnostics &amp; Logging
    </li>
</ul>


---

# Business Logic


---


# User Interface


Bla


---


# Storage


<ul>
  <li>
    Store data for repeated usage
  </li>
  <li>
    Some storage options:
    <ul>
      <li>Shared preferences</li>
      <li>Files</li>
      <li>Database</li>
    </ul>
  </li>
</ul>

+++

# Bla

---

# Networking

Bla

---


# Realm Mobile Platform




---

# Demo: Example Code

## User Registration

```
/*
 * Copyright 2016 Realm Inc.
 */

...
quite imports omitted here
...

public class RegisterActivity extends AppCompatActivity implements SyncUser.Callback {

    private AutoCompleteTextView usernameView;
    private EditText passwordView;
    private EditText passwordConfirmationView;
    private View progressView;
    private View registerFormView;
    private FacebookAuth facebookAuth;
    private GoogleAuth googleAuth;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_register);
        usernameView = (AutoCompleteTextView) findViewById(R.id.username);
        passwordView = (EditText) findViewById(R.id.password);
        passwordConfirmationView = (EditText) findViewById(R.id.password_confirmation);

        final Button mailRegisterButton = (Button) findViewById(R.id.email_register_button);
        mailRegisterButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                attemptRegister();
            }
        });

        registerFormView = findViewById(R.id.register_form);
        progressView = findViewById(R.id.register_progress);

        // Setup Facebook Authentication
        facebookAuth = new FacebookAuth((LoginButton) findViewById(R.id.login_button)) {
            @Override
            public void onRegistrationComplete(final LoginResult loginResult) {
                UserManager.setAuthMode(UserManager.AUTH_MODE.FACEBOOK);
                SyncCredentials credentials = SyncCredentials.facebook(loginResult.getAccessToken().getToken());
                SyncUser.loginAsync(credentials, AUTH_URL, RegisterActivity.this);
            }
        };

        // Setup Google Authentication
        googleAuth = new GoogleAuth((SignInButton) findViewById(R.id.sign_in_button), this) {
            @Override
            public void onRegistrationComplete(GoogleSignInResult result) {
                UserManager.setAuthMode(UserManager.AUTH_MODE.GOOGLE);
                GoogleSignInAccount acct = result.getSignInAccount();
                SyncCredentials credentials = SyncCredentials.google(acct.getIdToken());
                SyncUser.loginAsync(credentials, AUTH_URL, RegisterActivity.this);
            }
        };
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        googleAuth.onActivityResult(requestCode, resultCode, data);
        facebookAuth.onActivityResult(requestCode, resultCode, data);
    }

    private void attemptRegister() {
        
        // Error checking on input fields omitted
        ...
        
        
        if (cancel) {
            focusView.requestFocus();
        } else {
            showProgress(true);
            SyncUser.loginAsync(SyncCredentials.usernamePassword(username, password, true), AUTH_URL, new SyncUser.Callback() {
                @Override
                public void onSuccess(SyncUser user) {
                    registrationComplete(user);
                }

                @Override
                public void onError(ObjectServerError error) {
                    showProgress(false);
                    String errorMsg;
                    switch (error.getErrorCode()) {
                        case EXISTING_ACCOUNT: errorMsg = "Account already exists"; break;
                        default:
                            errorMsg = error.toString();
                    }
                    Toast.makeText(RegisterActivity.this, errorMsg, Toast.LENGTH_LONG).show();
                }
            });
        }
    }

    private void registrationComplete(SyncUser user) {
        UserManager.setActiveUser(user);
        Intent intent = new Intent(this, SignInActivity.class);
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
        startActivity(intent);
    }

}

```

+++

# Demo: Realm Object Server

<a target="_blank" href="http://ros-test.spinpos.com:9080/">Realm Object Server</a>




+++

# (Additional) Resources

<ul>
    <li><a href="https://realm.io/products/realm-mobile-platform/" target="_blank">Realm Mobile Platform</a></li>
    <li><a href="https://github.com/realm/realm-java/tree/master/examples" target="_blank">Realm Examples</a></li>
    <li><a href="https://github.com/realm-demos" target="_blank">Realm Demos</a></li>
    <li><a href="https://www.extendas.com" target="_blank">Extendas</a></li>
    <li><a href="https://www.extendas.com/corporate/vacatures/" target="_blank">Vacatures</a></li>
</ul>
