# sso
PHP单点登录解决方案,基于ThinkPHP

## sso中心配置
sso模块的下的 SSOServer类控制器
```php
  /**
     * 站点ID和钥密配置
     * Registered brokers
     * @var array
     */
    private static $brokers = [
        'Alice'   => ['secret'=>'8iwzik1bwd'], 
        'Greg'    => ['secret'=>'7pypoox2pc'],
        'Julias'  => ['secret'=>'ceda63kmhp']
    ];
```
### 站点配置
模块->类->方法配置
```php
class User
{
    
    public function index() {
        if (input('sso_error')) {
            return redirect('user/error', ['sso_error' => input('sso_error')], 307);
            //header("Location: error.php?sso_error=" . $_GET['sso_error'], true, 307);
            //exit;
        }
        // SSO授权中心地址
        $SSO_SERVER = "http://127.0.0.50/sso/index/index";
        // SSO站点配置里面的ID和钥密
        $SSO_BROKER_ID = "Alice"; 
        $SSO_BROKER_SECRET = "8iwzik1bwd";
        
        $broker = new \Jasny\SSO\Broker($SSO_SERVER, $SSO_BROKER_ID, $SSO_BROKER_SECRET);
        $broker->attach(true);
        $user = null;
        try {
            if (!empty(input('logout'))) {
                $broker->logout();
                $user = null;
            }else{
                $user = $broker->getUserInfo();
            }
        } catch (NotAttachedException $e) {
            header('Location: ' . $_SERVER['REQUEST_URI']);
            exit;
        } catch (SsoException $e) {
            return redirect('user/error', ['sso_error' => urlencode($e->getMessage())], 307);
            //header("Location: error.php?sso_error=" . $e->getMessage(), true, 307);
        }

        if (!$user) {
            return redirect('user/login', 307);
            //header("Location: login.php", true, 307);
            exit;
        }
        return view('index', ['user' => $user]);
    }
    
    
    public function login()
    {
        // SSO授权中心地址
        $SSO_SERVER = "http://127.0.0.50/sso/index/index";
        // SSO站点配置里面的ID和钥密
        $SSO_BROKER_ID = "Alice"; 
        $SSO_BROKER_SECRET = "8iwzik1bwd";
        
        $errmsg = '';
        
        $broker = new \Jasny\SSO\Broker($SSO_SERVER, $SSO_BROKER_ID, $SSO_BROKER_SECRET);
        $broker->attach(true);
        $user = $broker->getUserInfo();
        if($user){
            return redirect('user/index');
        }
        $request = request();
        if($request->isPOST()){
            //var_dump($broker->login(input('username'), input('password')));exit;
            try {
                if (!empty(input('logout'))) {
                    $broker->logout();
                } elseif ($user = $broker->login(input('username'), input('password'))) {
                    //header("Location: index.php", true, 303);
                    var_dump($user);exit;
                    exit;
                }
                
                $errmsg = "Login failed";
            } catch (NotAttachedException $e) {
                $errmsg = "Login failed";
                //header('Location: ' . $_SERVER['REQUEST_URI']);
                //exit;
            } catch (SsoException $e) {
                $errmsg = $e->getMessage();
            }
        }
        return view('login',['errmsg' => $errmsg]);

    }
}
```
