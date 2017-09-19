# REST in Peace

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
        <li>Since August 2016</li>
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

# Demo

<ul>
    <li>Some example Android code</li>
    <li>Realm Object Server backend</li>
    <li>Example application</li>
</ul>

+++


# Demo: Realm Tasks

![RealmTasks](https://raw.githubusercontent.com/realm-demos/realm-tasks/master/screenshot.jpg)


## User Registration

```

... Omitted some imports

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

## User Management

```

... Removed some imports

public class UserManager {
    // Supported authentication mode
    public enum AUTH_MODE {
        PASSWORD,
        FACEBOOK,
        GOOGLE
    }
    private static AUTH_MODE mode = AUTH_MODE.PASSWORD; // default

    public static void setAuthMode(AUTH_MODE m) {
        mode = m;
    }

    public static void logoutActiveUser() {
        switch (mode) {
            case PASSWORD: {
                // Do nothing, handled by the `User.currentUser().logout();`
                break;
            }
            case FACEBOOK: {
                LoginManager.getInstance().logOut();
                break;
            }
            case GOOGLE: {
                // the connection is handled by `enableAutoManage` mode
                break;
            }
        }
        SyncUser.currentUser().logout();
    }

    // Configure Realm for the current active user
    public static void setActiveUser(SyncUser user) {
        SyncConfiguration defaultConfig = new SyncConfiguration.Builder(user, RealmTasksApplication.REALM_URL).build();
        Realm.setDefaultConfiguration(defaultConfig);
    }
}

```

+++

## User Login


```

... Removed quite a number of imports

public class SignInActivity extends AppCompatActivity implements SyncUser.Callback {

    public static final String ACTION_IGNORE_CURRENT_USER = "action.ignoreCurrentUser";

    private AutoCompleteTextView usernameView;
    private EditText passwordView;
    private View progressView;
    private View loginFormView;
    private FacebookAuth facebookAuth;
    private GoogleAuth googleAuth;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        
        // Quite similar to the RegisterActivity.onCreate
        ...
        
    }

    @Override
    public void onSuccess(SyncUser user) {
        showProgress(false);
        loginComplete(user);
    }

    private void loginComplete(SyncUser user) {
        UserManager.setActiveUser(user);

        createInitialDataIfNeeded();

        Intent listActivity = new Intent(this, TaskListActivity.class);
        Intent tasksActivity = new Intent(this, TaskActivity.class);
        tasksActivity.putExtra(TaskActivity.EXTRA_LIST_ID, RealmTasksApplication.DEFAULT_LIST_ID);
        startActivities(new Intent[] { listActivity, tasksActivity} );
        finish();
    }

    private static void createInitialDataIfNeeded() {
        final Realm realm = Realm.getDefaultInstance();
        //noinspection TryFinallyCanBeTryWithResources
        try {
            if (realm.where(TaskListList.class).count() != 0) {
                return;
            }
            realm.executeTransaction(new Realm.Transaction() {
                @Override
                public void execute(Realm realm) {
                    if (realm.where(TaskListList.class).count() == 0) {
                        final TaskListList taskListList = realm.createObject(TaskListList.class, 0);
                        final TaskList taskList = new TaskList();
                        taskList.setId(RealmTasksApplication.DEFAULT_LIST_ID);
                        taskList.setText(RealmTasksApplication.DEFAULT_LIST_NAME);
                        taskListList.getItems().add(taskList);
                    }
                }
            });
        } finally {
            realm.close();
        }
    }
}

```

+++

## Showing and updating data

``` 

public class TaskListActivity extends AppCompatActivity {

    private Realm realm;
    private RecyclerViewWithEmptyViewSupport recyclerView;
    private TaskListAdapter adapter;
    private TouchHelper touchHelper;
    private RealmResults<TaskListList> list;
    private boolean logoutAfterClose;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_common_list);
        recyclerView = (RecyclerViewWithEmptyViewSupport) findViewById(R.id.recycler_view);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        recyclerView.setEmptyView(findViewById(R.id.empty_view));
    }

    @Override
    protected void onStart() {
        super.onStart();
        if (touchHelper != null) {
            touchHelper.attachToRecyclerView(null);
        }
        adapter = null;
        realm = Realm.getDefaultInstance();
        list = realm.where(TaskListList.class).findAll();
        list.addChangeListener(new RealmChangeListener<RealmResults<TaskListList>>() {
            @Override
            public void onChange(RealmResults<TaskListList> results) {
                updateList(results);
            }
        });
        updateList(list);
    }

    private void updateList(RealmResults<TaskListList> results) {

        if (results.size() > 0 && adapter == null) {

            // Code omitted for brevity

            // Create Adapter
            adapter = new TaskListAdapter(TaskListActivity.this, results.first().getItems());
            touchHelper = new TouchHelper(new Callback(), adapter);
            touchHelper.attachToRecyclerView(recyclerView);
        }
    }

    @Override
    protected void onStop() {
        list.removeChangeListeners();
        if (adapter != null) {
            touchHelper.attachToRecyclerView(null);
            adapter = null;
        }
        realm.removeAllChangeListeners();
        realm.close();
        realm = null;
        if (logoutAfterClose) {
            /*
             * We need call logout() here since onCreate() of the next Activity is already
             * executed before reaching here.
             */
            UserManager.logoutActiveUser();
            logoutAfterClose = false;
        }
        super.onStop();
    }

    private class Callback implements TouchHelper.Callback {

        @Override
        public void onMoved(RecyclerView recyclerView, ItemViewHolder from, ItemViewHolder to) {
            final int fromPosition = from.getAdapterPosition();
            final int toPosition = to.getAdapterPosition();
            adapter.onItemMoved(fromPosition, toPosition);
            adapter.notifyItemMoved(fromPosition, toPosition);
        }

        @Override
        public void onCompleted(ItemViewHolder viewHolder) {
            adapter.onItemCompleted(viewHolder.getAdapterPosition());
            adapter.notifyDataSetChanged();
        }

        @Override
        public void onDismissed(ItemViewHolder viewHolder) {
            final int position = viewHolder.getAdapterPosition();
            adapter.onItemDismissed(position);
            adapter.notifyItemRemoved(position);
        }

        @Override
        public boolean onClicked(ItemViewHolder viewHolder) {
            final int position = viewHolder.getAdapterPosition();
            final TaskList taskList = adapter.getItem(position);
            final String id = taskList.getId();
            final Intent intent = new Intent(TaskListActivity.this, TaskActivity.class);
            intent.putExtra(TaskActivity.EXTRA_LIST_ID, id);
            TaskListActivity.this.startActivity(intent);
            return true;
        }

        @Override
        public void onChanged(ItemViewHolder viewHolder) {
            adapter.onItemChanged(viewHolder);
            adapter.notifyItemChanged(viewHolder.getAdapterPosition());
        }

        @Override
        public void onAdded() {
            adapter.onItemAdded();
            adapter.notifyItemInserted(0);
        }

    }
}

public class TaskListAdapter extends CommonAdapter<TaskList> implements TouchHelperAdapter {

    public TaskListAdapter(Context context, OrderedRealmCollection<TaskList> items) {
        super(context, items);
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        super.onBindViewHolder(holder, position);
        final ItemViewHolder itemViewHolder = (ItemViewHolder) holder;
        final TaskList taskList = getItem(position);
        itemViewHolder.getText().setText(taskList.getText());
        itemViewHolder.setBadgeVisible(true);
        final long badgeCount = taskList.getItems().where().equalTo(TaskList.FIELD_COMPLETED, false).count();
        itemViewHolder.setBadgeCount((int) badgeCount);
        itemViewHolder.setCompleted(taskList.isCompleted());
    }

    @Override
    public void onItemAdded() {
        final Realm realm = Realm.getDefaultInstance();
        realm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                final TaskList taskList = new TaskList();
                taskList.setId(UUID.randomUUID().toString());
                taskList.setText("");
                getData().add(0, taskList);
            }
        });
        realm.close();
    }

    @Override
    public void onItemMoved(final int fromPosition, final int toPosition) {
        final Realm realm = Realm.getDefaultInstance();
        realm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                moveItems(fromPosition, toPosition);
            }
        });
        realm.close();
    }

    @Override
    public void onItemCompleted(final int position) {
        final TaskList taskList = getItem(position);
        final Realm realm = Realm.getDefaultInstance();
        final int count = (int) getData().where().equalTo(TaskList.FIELD_COMPLETED, false).count();
        realm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                if (!taskList.isCompleted()) {
                    if (taskList.isCompletable()) {
                        taskList.setCompleted(true);
                        moveItems(position, count - 1);
                    } else {
                        Toast.makeText(context, R.string.no_item, Toast.LENGTH_SHORT).show();
                    }
                } else {
                    taskList.setCompleted(false);
                    moveItems(position, count);
                }
            }
        });
        realm.close();
    }

    @Override
    public void onItemChanged(final ItemViewHolder viewHolder) {
        final Realm realm = Realm.getDefaultInstance();
        final int position = viewHolder.getAdapterPosition();

        realm.executeTransaction(new Realm.Transaction() {
            @Override
            public void execute(Realm realm) {
                TaskList taskList = getItem(position);
                taskList.setText(viewHolder.getText().getText().toString());
            }
        });
        realm.close();
    }
}

```


+++

# Demo: Realm Object Server

Realm Object Server Developer Edition is free to use!*

Paid options are available for more functions.

<a target="_blank" href="http://ros-test.spinpos.com:9080/">Realm Object Server</a>

<h6>*Free as in free beer</h6>


+++

# Demo: Drawing App

<ul>
    <li>Real-time sharing of drawings</li>
    <li>Android and iOS can draw together</li>
    <li>Available at <a target="_blank" href="https://github.com/realm-demos/realm-draw">GitHub</a></li>
</ul>

+++?image=http://i.giphy.com/90F8aUepslB84.gif

# Such Wow!


---

# (Additional) Resources

<ul>
    <li><a href="https://realm.io/products/realm-mobile-platform/" target="_blank">Realm Mobile Platform</a></li>
    <li><a href="https://github.com/realm/realm-java/tree/master/examples" target="_blank">Realm Examples</a></li>
    <li><a href="https://github.com/realm-demos" target="_blank">Realm Demos</a></li>
    <li><a href="https://www.extendas.com" target="_blank">Extendas</a></li>
    <li><a href="https://www.extendas.com/corporate/vacatures/" target="_blank">Vacatures</a></li>
</ul>
