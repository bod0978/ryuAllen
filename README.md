# ryuAllen
# ryu-allen-codes
# //The customer environment requires the user to directly enter the account password to use//


public class RegisterActivity extends Activity {

    private ProgressDialog dialog;
    Vphone Vphone;
    
   
    
    public static String RegDomain = "192.168.100.9";//test service ip port 5060;//" //
    public static String RegPassword = "";
    public static String RegUser = "";


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_register);
        try {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                Intent intent = new Intent();
                intent.setAction(Settings.ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS);
                intent.setData(Uri.parse("package:" + getPackageName()));
                startActivity(intent);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        // Get Vphone instance
        Vphone = ((VApplication) getApplication()).getVphone();

        Button regButton = (Button) findViewById(R.id.register_button);
        final EditText userEdit = (EditText) findViewById(R.id.login);
        final EditText passEdit = (EditText) findViewById(R.id.password);
        //final EditText domainEdit = (EditText) findViewById(R.id.domain);
        final CheckBox disableReg =(CheckBox) findViewById(R.id.disable_registration);

        ///test
        userEdit.setText(RegUser);
        passEdit.setText(RegPassword);
        //domainEdit.setText(RegDomain);

        regButton.setOnClickListener(new View.OnClickListener() {

            public void onClick(View v) {

                //Show progress
                if(dialog==null)
                {
                    dialog = new ProgressDialog(RegisterActivity.this);
                    dialog.setCancelable(false);
                    dialog.setMessage("Registering...");
                    dialog.setCancelable(false);
                }

                dialog.show();

                int regTimeout = disableReg.isChecked() ? 0 : 300;

                RegUser = userEdit.getText().toString();
                RegPassword = passEdit.getText().toString();
                //RegDomain = domainEdit.getText().toString();

                // Add account
                //Vphone.getConfig().setContactDetailsUri("");
                //Vphone.getConfig().setContactDetails(";token="+URLEncoder.encode("yyyy+xxx 111 <l>"));
                long accId = Vphone.getConfig().addAccount(RegDomain, null, RegUser, RegPassword, null, "", regTimeout, false);

                //Register
                try {
                    Vphone.register();
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });


        // Set registration event
        Vphone.setRegistrationStateListener(new OnRegistrationListener() {

            public void onRegistrationFailed(long accId, int statusCode, String statusText) {

                if(dialog != null) dialog.dismiss();

                AlertDialog.Builder fail = new AlertDialog.Builder(RegisterActivity.this);
                fail.setTitle("Registration failed");
                fail.setMessage(statusCode + " - " + statusText);
                fail.setPositiveButton("Ok", new DialogInterface.OnClickListener() {

                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                });

                fail.show();
            }

            public void onRegistered(long accId) {

                //Hide progress
                if(dialog != null) dialog.dismiss();

                //Unsubscribe reg events
                Vphone.setRegistrationStateListener(null);

                //Start main activity
                Intent intent = new Intent(RegisterActivity.this, MainActivity.class);
                startActivity(intent);

                //Close this activity
                finish();
            }

            @Override
            public void onUnRegistered(long accId) {
                Log.e(this.toString(), "log succ acc = " + accId);

                Toast.makeText(RegisterActivity.this, "RegisterActivity::onUnRegistered", Toast.LENGTH_SHORT).show();
            }
        }); //registration listener

    }//onCreate


    @Override
    public void onBackPressed() {
        //Exit app here
        try {
            Vphone.setRegistrationStateListener(null);

            //Destroy phone
            if(Vphone.isActive())        Vhone.destroy();

        } catch (RemoteException e) {
            e.printStackTrace();
        }
        super.onBackPressed();
    }

}

