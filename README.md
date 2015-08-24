# VAPP
The VAPP! mobile payment SDK for Android

# Getting Started
VAPP! is provided as a standalone [aar file](http://tools.android.com/tech-docs/new-build-system/aar-format).
You should copy this file to **/MyProject/app/aars/vapp.aar**. Your build.gradle file should then be modified 
so that it matches the code snippet below:

```
apply plugin: 'com.android.application'

android {
    repositories { // make gradle search the app/aars directory for source sets
        flatDir {
            dirs 'aars'
        }
    }
}

dependencies {
    compile(name:'vapp', ext:'aar') // add VAPP! SDK as a dependency
}
```

After syncing your gradle files, your project should be ready to initialise VAPP! with a list of products.
See the [sample application](https://github.com/vasilitate/VAPP-Store) if you are having trouble with this step.

### Initialising VAPP!
Vapp.initialise() must be called before any SMS payments are made. Initialisation requires several pieces of information:

- Your app id, which identifies your payments
- The number range which you have been assigned
- A list of products within your app, which can be paid for using SMS

It is recommended that you initialise VAPP! within a custom [Application class](http://developer.android.com/reference/android/app/Application.html),
using something similar to the snippet below:

```
private static final String VAPP_APP_ID = "MyAppId";

List<VappProduct> productList = new ArrayList<>();
VappNumberRange vappNumberRange = new VappNumberRange("+447458830000", "+447458730009");

Vapp.initialise(this, VAPP_APP_ID, productList, vappNumberRange, false);
```

### Defining Products
Any products within your app must be passed into the SDK during initialisation. A VappProduct requires a unique alphabetic ID, the number of SMS messages which will be sent, and the maximum number of times that the product can be purchased.

```
private static final String MY_PRODUCT_ID = "MyProduct";
private static final int SMS_COUNT = 15;
private static final int MAX_PURCHASE_COUNT = 1;

List<VappProduct> productList = new ArrayList<>();
productList.add(new VappProduct(MY_PRODUCT_ID, SMS_COUNT, MAX_PURCHASE_COUNT);
Vapp.initialise(this, VAPP_APP_ID, productList, vappNumberRange, false);
```

### Purchasing Products

Products can be purchased by calling **showVappPaymentScreen()**. Before initiating the sale, you should indicate to the user what they are buying and how much it will cost, like below:

```
new AlertDialog.Builder(PaymentActivity.this)
      .setTitle("Confirm Payment")
      .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
          @Override public void onClick(DialogInterface dialog, int which) {
            Vapp.showVappPaymentScreen(context, MY_PRODUCT, false);
          }
      }).show();
```

Only one product may be purchased at a time.  The SDK will enforce this however you should design your app so that it is obvious to the user that this is the case, e.g. by disabling purchasing options while one is in progress.

### Tracking Purchases

The SMS purchase may take some time (10's of minutes!), callbacks are provided with the VappProgressReceiver:
```
@Override public void onCreate(@Nullable Bundle savedInstanceState) {
  super.onCreate(savedInstanceState)
  VappProgressReceiver vappProgressReceiver = new VappProgressReceiver(context, listener);
  vappProgressReceiver.onCreate();
}

@Override public void onDestroy() {
    super.onDestroy();
    if (vappProgressReceiver != null) {
        vappProgressReceiver.onDestroy();
        vappProgressReceiver = null;
    }
}
```

### Progress Widget ###

You can make use of a purpose built progress view which allows you to display either the number of SMS's sent or the percentage completion as a progress bar and/or a numeric.  Just add the following to your layout file:

...
    <com.vasilitate.vapp.sdk.VappProgressWidget
        android:id="@+id/progress_widget"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
...

'''
    VappProgressWidget progressWidget = (VappProgressWidget) findViewById( R.id.progress_widget);
 
    vappProgressReceiver = new VappProgressReceiver(this, progressWidget);
    vappProgressReceiver.onCreate();
'''



It is also possible to determine whether a product has been purchased using synchronous methods. Your app can then allow the user access to purchased content, if VAPP! has recorded the product as purchased.

```
boolean hasPaid = Vapp.isPaidFor(context, MY_PRODUCT));
boolean isPaying = Vapp.getSMSPaymentProgress(context, MY_PRODUCT) > 0
```
