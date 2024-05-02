<center>

# STUDI KASUS LARAVEL DASAR

</center>

<center>

## POINT UTAMA

</center>

### 1. Membuat project

-   Minimal versi `PHP` ada versi 8,

-   Dan composer versi 2 atau lebih,

-   Pada cmd gunakan perintah `composer create-project laravel/laravel=9.15 studi-kasus-laravel-dasar`.

---

### 2. Membuat logic login

-   Dalam studi kasus ini, saya tidak menggunakan database untuk menyimpan user login dan todo,

-   Dan untuk logic login nya diganti dengan menggunakan dengan menggunakan `array` untuk user login.

    ```PHP
     private array $users = [
        "akbar" => "rahasia" // username & pass
    ];

    // function untuk login
    function login(string $user, string $password): bool
    {
        if(!isset($this->users[$user])){
            return false;
        }

        $correctPassword = $this->users[$user];
        return $password == $correctPassword;
    }
    ```

-   Unit test login

    ```PHP
    public function testLoginSuccess() //ketika login berhasil
    {
        $this->post('/login', [
            "user" => "akbar",
            "password" => "rahasia"
        ])->assertRedirect("/")
            ->assertSessionHas("user", "akbar");
    }
    ```

---

### 3. Template halaman web

-   Template halaman web berada didalam directory `resources/view/`.

-   Test template

    ```PHP
    Route::view('/template', 'template');
    ```

---

### 4. Membuat user controller

-   Untuk memanggil halaman login & logout.

-   Function User login benar

    ```PHP
      public function login(): Response
    {
        return response()
            ->view("user.login", [
                "title" => "Login"
            ]);
    }
    ```

-   Function user login ketika salah

    ```PHP
    public function doLogin(Request $request): Response|RedirectResponse
    {
        $user = $request->input('user');
        $password = $request->input('password');

        // validate input
        if (empty($user) || empty($password)) {
            return response()->view("user.login", [
                "title" => "Login",
                "error" => "User or password is required"
            ]);
        }

        if ($this->userService->login($user, $password)) {
            $request->session()->put("user", $user);
            return redirect("/");
        }

        return response()->view("user.login", [
            "title" => "Login",
            "error" => "User or password is wrong"
        ]);
    }
    ```

-   Fuction user logout

    ```PHP
    public function doLogout(Request $request): RedirectResponse
    {
        $request->session()->forget("user");
        return redirect("/");
    }
    ```

-   Menampilkan halaman login & logout

    ```PHP
    Route::controller(\App\Http\Controllers\UserController::class)->group(function () {
    Route::get('/login', 'login')->middleware([\App\Http\Middleware\OnlyGuestMiddleware::class]);
    Route::post('/login', 'doLogin')->middleware([\App\Http\Middleware\OnlyGuestMiddleware::class]);
    Route::post('/logout', 'doLogout')->middleware([\App\Http\Middleware\OnlyMemberMiddleware::class]);
    });
    ```
