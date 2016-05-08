# 前言
几乎每个app都会有登录注册的功能，可以看看笔者开发的『南方周末新闻阅读器』，登录、手机注册、忘记密码这些入口，这些功能在app中要如何来实现呢？这个模块看似很简单，但要做好就需要考虑很多细节，比如对用户的输入的容错，操作的提示文案的设定，登录成功保存用户信息等等。

# 业务流程图

![登录注册流程](http://img.blog.csdn.net/20160124214550575)


# 业务逻辑描述
上一节的流程图已经很清晰的展现了登录注册的流程，这里继续用文字说明一下：
1. 点击进入个人中心或者需要用户登录状态的操作，先判断用户是否已经登录。
2. 如果已经登录，则继续后面的业务，否则，跳转到登录页面进行登录。
3. 如果已经有账号，则可以直接登录，或者可以直接选择第三方平台授权登录。
4. 如果未注册账号，则需要先进行账号注册，注册成功后再登录；也可以不注册账号，通过第三方平台授权进行登录。
5. 如果有账号，但忘记密码，则需要进行重置密码，否则直接登录。

# 具体实现
登录可以使用账号登录，现在的app基本上都是手机号码登录，注册的时候也是一个手机对应一个账号，通过发送验证码进行验证；用户也可以选择第三方平台进行登录，一般会提供微信、QQ、新浪微博这样的主流社交平台进行授权登录，这里笔者使用了**友盟的SDK**进行实现。

示例代码：LoginActivity.java
```
package com.devilwwj.loginandregister;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.text.TextUtils;
import android.text.method.HideReturnsTransformationMethod;
import android.text.method.PasswordTransformationMethod;
import android.view.KeyEvent;
import android.view.View;
import android.view.inputmethod.EditorInfo;
import android.widget.TextView;
import android.widget.Toast;

import com.devilwwj.loginandregister.global.AppConstants;
import com.devilwwj.loginandregister.utils.LogUtils;
import com.devilwwj.loginandregister.utils.ProgressDialogUtils;
import com.devilwwj.loginandregister.utils.RegexUtils;
import com.devilwwj.loginandregister.utils.ShareUtils;
import com.devilwwj.loginandregister.utils.SpUtils;
import com.devilwwj.loginandregister.utils.ToastUtils;
import com.devilwwj.loginandregister.utils.Utils;
import com.devilwwj.loginandregister.views.CleanEditText;
import com.umeng.socialize.bean.SHARE_MEDIA;
import com.umeng.socialize.controller.UMServiceFactory;
import com.umeng.socialize.controller.UMSocialService;
import com.umeng.socialize.controller.listener.SocializeListeners.UMAuthListener;
import com.umeng.socialize.controller.listener.SocializeListeners.UMDataListener;
import com.umeng.socialize.exception.SocializeException;

import java.util.Map;

import static android.view.View.OnClickListener;

/**
 * @desc 登录界面
 * Created by devilwwj on 16/1/24.
 */
public class LoginActivity extends Activity implements OnClickListener {

    private static final String TAG = "loginActivity";
    private static final int REQUEST_CODE_TO_REGISTER = 0x001;

    // 界面控件
    private CleanEditText accountEdit;
    private CleanEditText passwordEdit;

    // 第三方平台获取的访问token，有效时间，uid
    private String accessToken;
    private String expires_in;
    private String uid;
    private String sns;

    // 整个平台的Controller，负责管理整个SDK的配置、操作等处理
    private UMSocialService mController = UMServiceFactory
            .getUMSocialService(AppConstants.DESCRIPTOR);


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        initViews();

        // 配置分享平台
        ShareUtils.configPlatforms(this);
    }

    /**
     * 初始化视图
     */
    private void initViews() {
        accountEdit = (CleanEditText) this.findViewById(R.id.et_email_phone);
        accountEdit.setImeOptions(EditorInfo.IME_ACTION_NEXT);
        accountEdit.setTransformationMethod(HideReturnsTransformationMethod
                .getInstance());
        passwordEdit = (CleanEditText) this.findViewById(R.id.et_password);
        passwordEdit.setImeOptions(EditorInfo.IME_ACTION_DONE);
        passwordEdit.setImeOptions(EditorInfo.IME_ACTION_GO);
        passwordEdit.setTransformationMethod(PasswordTransformationMethod
                .getInstance());
        passwordEdit.setOnEditorActionListener(new TextView.OnEditorActionListener() {

            @Override
            public boolean onEditorAction(TextView v, int actionId,
                                          KeyEvent event) {
                if (actionId == EditorInfo.IME_ACTION_DONE
                        || actionId == EditorInfo.IME_ACTION_GO) {
                    clickLogin();
                }
                return false;
            }
        });
    }

    private void clickLogin() {
        String account = accountEdit.getText().toString();
        String password = passwordEdit.getText().toString();
        if (checkInput(account, password)) {
            // TODO: 请求服务器登录账号
        }
    }

    /**
     * 检查输入
     *
     * @param account
     * @param password
     * @return
     */
    public boolean checkInput(String account, String password) {
        // 账号为空时提示
        if (account == null || account.trim().equals("")) {
            Toast.makeText(this, R.string.tip_account_empty, Toast.LENGTH_LONG)
                    .show();
        } else {
            // 账号不匹配手机号格式（11位数字且以1开头）
            if ( !RegexUtils.checkMobile(account)) {
                Toast.makeText(this, R.string.tip_account_regex_not_right,
                        Toast.LENGTH_LONG).show();
            } else if (password == null || password.trim().equals("")) {
                Toast.makeText(this, R.string.tip_password_can_not_be_empty,
                        Toast.LENGTH_LONG).show();
            } else {
                return true;
            }
        }

        return false;
    }

    @Override
    public void onClick(View v) {
        Intent intent = null;
        switch (v.getId()) {
            case R.id.iv_cancel:
                finish();
                break;
            case R.id.btn_login:
                clickLogin();
                break;
            case R.id.iv_wechat:
                clickLoginWexin();
                break;
            case R.id.iv_qq:
                clickLoginQQ();
                break;
            case R.id.iv_sina:
                loginThirdPlatform(SHARE_MEDIA.SINA);
                break;
            case R.id.tv_create_account:
                enterRegister();
                break;
            case R.id.tv_forget_password:
                enterForgetPwd();
                break;
            default:
                break;
        }
    }

    /**
     * 点击使用QQ快速登录
     */
    private void clickLoginQQ() {
        if (!Utils.isQQClientAvailable(this)) {
            ToastUtils.showShort(LoginActivity.this,
                    getString(R.string.no_install_qq));
        } else {
            loginThirdPlatform(SHARE_MEDIA.QZONE);
        }
    }

    /**
     * 点击使用微信登录
     */
    private void clickLoginWexin() {
        if (!Utils.isWeixinAvilible(this)) {
            ToastUtils.showShort(LoginActivity.this,
                    getString(R.string.no_install_wechat));
        } else {
            loginThirdPlatform(SHARE_MEDIA.WEIXIN);
        }
    }

    /**
     * 跳转到忘记密码
     */
    private void enterForgetPwd() {
        Intent intent = new Intent(this, ForgetPasswordActivity.class);
        startActivity(intent);
    }

    /**
     * 跳转到注册页面
     */
    private void enterRegister() {
        Intent intent = new Intent(this, SignUpActivity.class);
        startActivityForResult(intent, REQUEST_CODE_TO_REGISTER);
    }


    /**
     * 授权。如果授权成功，则获取用户信息
     *
     * @param platform
     */
    private void loginThirdPlatform(final SHARE_MEDIA platform) {
        mController.doOauthVerify(LoginActivity.this, platform,
                new UMAuthListener() {

                    @Override
                    public void onStart(SHARE_MEDIA platform) {
                        LogUtils.i(TAG, "onStart------"
                                + Thread.currentThread().getId());
                        ProgressDialogUtils.getInstance().show(
                                LoginActivity.this,
                                getString(R.string.tip_begin_oauth));
                    }

                    @Override
                    public void onError(SocializeException e,
                                        SHARE_MEDIA platform) {
                        LogUtils.i(TAG, "onError------"
                                + Thread.currentThread().getId());
                        ToastUtils.showShort(LoginActivity.this,
                                getString(R.string.oauth_fail));
                        ProgressDialogUtils.getInstance().dismiss();
                    }

                    @Override
                    public void onComplete(Bundle value, SHARE_MEDIA platform) {
                        LogUtils.i(TAG, "onComplete------" + value.toString());
                        if (platform == SHARE_MEDIA.SINA) {
                            accessToken = value.getString("access_key");
                        } else {
                            accessToken = value.getString("access_token");
                        }
                        expires_in = value.getString("expires_in");
                        // 获取uid
                        uid = value.getString(AppConstants.UID);
                        if (value != null && !TextUtils.isEmpty(uid)) {
                            // uid不为空，获取用户信息
                            getUserInfo(platform);
                        } else {
                            ToastUtils.showShort(LoginActivity.this,
                                    getString(R.string.oauth_fail));
                        }
                    }

                    @Override
                    public void onCancel(SHARE_MEDIA platform) {
                        LogUtils.i(TAG, "onCancel------"
                                + Thread.currentThread().getId());
                        ToastUtils.showShort(LoginActivity.this,
                                getString(R.string.oauth_cancle));
                        ProgressDialogUtils.getInstance().dismiss();

                    }
                });
    }


    /**
     * 获取用户信息
     *
     * @param platform
     */
    private void getUserInfo(final SHARE_MEDIA platform) {
        mController.getPlatformInfo(LoginActivity.this, platform,
                new UMDataListener() {

                    @Override
                    public void onStart() {
                        // 开始获取
                        LogUtils.i("getUserInfo", "onStart------");
                        ProgressDialogUtils.getInstance().dismiss();
                        ProgressDialogUtils.getInstance().show(
                                LoginActivity.this, "正在请求...");
                    }

                    @Override
                    public void onComplete(int status, Map<String, Object> info) {

                        try {
                            String sns_id = "";
                            String sns_avatar = "";
                            String sns_loginname = "";
                            if (info != null && info.size() != 0) {
                                LogUtils.i("third login", info.toString());

                                if (platform == SHARE_MEDIA.SINA) { // 新浪微博
                                    sns = AppConstants.SINA;
                                    if (info.get(AppConstants.UID) != null) {
                                        sns_id = info.get(AppConstants.UID)
                                                .toString();
                                    }
                                    if (info.get(AppConstants.PROFILE_IMAGE_URL) != null) {
                                        sns_avatar = info
                                                .get(AppConstants.PROFILE_IMAGE_URL)
                                                .toString();
                                    }
                                    if (info.get(AppConstants.SCREEN_NAME) != null) {
                                        sns_loginname = info.get(
                                                AppConstants.SCREEN_NAME)
                                                .toString();
                                    }
                                } else if (platform == SHARE_MEDIA.QZONE) { // QQ
                                    sns = AppConstants.QQ;
                                    if (info.get(AppConstants.UID) == null) {
                                        ToastUtils
                                                .showShort(
                                                        LoginActivity.this,
                                                        getString(R.string.oauth_fail));
                                        return;
                                    }
                                    sns_id = info.get(AppConstants.UID)
                                            .toString();
                                    sns_avatar = info.get(
                                            AppConstants.PROFILE_IMAGE_URL)
                                            .toString();
                                    sns_loginname = info.get(
                                            AppConstants.SCREEN_NAME)
                                            .toString();
                                } else if (platform == SHARE_MEDIA.WEIXIN) { // 微信
                                    sns = AppConstants.WECHAT;
                                    sns_id = info.get(AppConstants.OPENID)
                                            .toString();
                                    sns_avatar = info.get(
                                            AppConstants.HEADIMG_URL)
                                            .toString();
                                    sns_loginname = info.get(
                                            AppConstants.NICKNAME).toString();
                                }

                                // 这里直接保存第三方返回来的用户信息
                                SpUtils.putBoolean(LoginActivity.this,
                                        AppConstants.THIRD_LOGIN, true);

                                LogUtils.e("info", sns + "," + sns_id + ","
                                        + sns_loginname);

                                // TODO: 这里执行第三方连接(绑定服务器账号）


                            }
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }

                });
    }
}

```

示例代码：SignUpActivity.java
```
package com.devilwwj.loginandregister;

import android.app.Activity;
import android.os.Bundle;
import android.text.TextUtils;
import android.util.Log;
import android.view.KeyEvent;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.inputmethod.EditorInfo;
import android.widget.Button;
import android.widget.TextView;
import android.widget.TextView.OnEditorActionListener;

import com.devilwwj.loginandregister.utils.RegexUtils;
import com.devilwwj.loginandregister.utils.ToastUtils;
import com.devilwwj.loginandregister.utils.VerifyCodeManager;
import com.devilwwj.loginandregister.views.CleanEditText;

/**
 * @desc 注册界面
 * 功能描述：一般会使用手机登录，通过获取手机验证码，跟服务器交互完成注册
 * Created by devilwwj on 16/1/24.
 */
public class SignUpActivity extends Activity implements OnClickListener{
    private static final String TAG = "SignupActivity";
    // 界面控件
    private CleanEditText phoneEdit;
    private CleanEditText passwordEdit;
    private CleanEditText verifyCodeEdit;
    private Button getVerifiCodeButton;

    private VerifyCodeManager codeManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_signup);

        initViews();
        codeManager = new VerifyCodeManager(this, phoneEdit, getVerifiCodeButton);

    }

    /**
     * 通用findViewById,减少重复的类型转换
     *
     * @param id
     * @return
     */
    @SuppressWarnings("unchecked")
    public final <E extends View> E getView(int id) {
        try {
            return (E) findViewById(id);
        } catch (ClassCastException ex) {
            Log.e(TAG, "Could not cast View to concrete class.", ex);
            throw ex;
        }
    }
    

    private void initViews() {

        getVerifiCodeButton = getView(R.id.btn_send_verifi_code);
        getVerifiCodeButton.setOnClickListener(this);
        phoneEdit = getView(R.id.et_phone);
        phoneEdit.setImeOptions(EditorInfo.IME_ACTION_NEXT);// 下一步
        verifyCodeEdit = getView(R.id.et_verifiCode);
        verifyCodeEdit.setImeOptions(EditorInfo.IME_ACTION_NEXT);// 下一步
        passwordEdit = getView(R.id.et_password);
        passwordEdit.setImeOptions(EditorInfo.IME_ACTION_DONE);
        passwordEdit.setImeOptions(EditorInfo.IME_ACTION_GO);
        passwordEdit.setOnEditorActionListener(new OnEditorActionListener() {

            @Override
            public boolean onEditorAction(TextView v, int actionId,
                                          KeyEvent event) {
                // 点击虚拟键盘的done
                if (actionId == EditorInfo.IME_ACTION_DONE
                        || actionId == EditorInfo.IME_ACTION_GO) {
                    commit();
                }
                return false;
            }
        });
    }

    private void commit() {
        String phone = phoneEdit.getText().toString().trim();
        String password = passwordEdit.getText().toString().trim();
        String code = verifyCodeEdit.getText().toString().trim();

        if (checkInput(phone, password, code)) {
            // TODO:请求服务端注册账号
        }
    }

    private boolean checkInput(String phone, String password, String code) {
        if (TextUtils.isEmpty(phone)) { // 电话号码为空
            ToastUtils.showShort(this, R.string.tip_phone_can_not_be_empty);
        } else {
            if (!RegexUtils.checkMobile(phone)) { // 电话号码格式有误
                ToastUtils.showShort(this, R.string.tip_phone_regex_not_right);
            } else if (TextUtils.isEmpty(code)) { // 验证码不正确
                ToastUtils.showShort(this, R.string.tip_please_input_code);
            } else if (password.length() < 6 || password.length() > 32
                    || TextUtils.isEmpty(password)) { // 密码格式
                ToastUtils.showShort(this,
                        R.string.tip_please_input_6_32_password);
            } else {
                return true;
            }
        }

        return false;
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn_send_verifi_code:
                // TODO 请求接口发送验证码
                codeManager.getVerifyCode(VerifyCodeManager.REGISTER);
                break;

            default:
                break;
        }
    }
}

```

# Github
登录注册的解决方案，笔者已经做成一个Demo放到github了，大家在实际开发的时候可以参考着根据自身的业务进行调整，但基本上不会差太多，第三方登录、验证码这个都可以选用第三方服务来实现，github地址如下：
[登录注册解决方案](https://github.com/fanatic-mobile-developer-for-android/A-week-to-develop-android-app-plan/tree/master/002_LoginAndRegisterModule)

>作者：巫山老妖
>博客地址：http://blog.csdn.net/wwj_748?viewmode=contents
>微信号：whatswwj
>微信公众号：wwjblog
