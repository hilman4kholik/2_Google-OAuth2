# Membuat login google dengan OAuth2 di golang
1.	Membuat akun google cloud di https://console.developers.google.com.
2.	Pilih menu APIs & Services.
3.	Buat dulu projectnya.
4.	Buat juga OAuth consent screen.
5.	Lalu pergi ke menu credentials -> Create Credentials -> OAtuh Client ID -> pilih jenis aplikasi yang akan digunakan.
6.	Bagian Authorized redirect URLs isikan http//localhost:8080/callback untuk test.
7.	Lalu Create.
8.	Akan muncul tampilan seperti ini
9.	Lalu buat folder project dan setting dengan git init.
mkdir 2_google-oauth2
git init
go mod init 2_google-oauth2
go get golang.org/x/oauth2
go get golang.org/x/oatuh2/google
10.	Buat file main.go
dengan isi seperti ini:
```
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"

	"golang.org/x/oauth2"
	"golang.org/x/oauth2/google"
)

var (
	googleOAuthConfig = &oauth2.Config{
		RedirectURL:  "http://localhost:8080/callback", //URL ini harus sama dengan redirect_uri pada google developer console
		ClientID:     "SISIPKAN CLIENT ID DISINI",
		ClientSecret: "SISIPKAN CLIENT SECRET DISINI",
		// scopes API URL bisa didapatkan di https://developers.google.com/identity/protocols/oauth2/scopes
		Scopes:   []string{"https://www.googleapis.com/auth/userinfo.profile"},
		Endpoint: google.Endpoint,
	}
	randomState = "random"
)

func main() {
	http.HandleFunc("/", handleHome)
	http.HandleFunc("/login", handleLogin)
	http.HandleFunc("/callback", handleCallback)
	http.ListenAndServe(":8080", nil)
}

func handleHome(w http.ResponseWriter, r *http.Request) {
	halaman := "/login"
	html := "<html><body><a href=" + halaman + ">Login using google</a></body></html>"
	fmt.Fprint(w, html)
}

func handleLogin(w http.ResponseWriter, r *http.Request) {
	url := googleOAuthConfig.AuthCodeURL(randomState)
	http.Redirect(w, r, url, http.StatusTemporaryRedirect)
}

func handleCallback(w http.ResponseWriter, r *http.Request) {
	if r.FormValue("state") != randomState {
		fmt.Println("State is not valid")
		http.Redirect(w, r, "/", http.StatusTemporaryRedirect)
		return
	}
	token, err := googleOAuthConfig.Exchange(oauth2.NoContext, r.FormValue("code"))
	if err != nil {
		fmt.Println("could not get token \n", err.Error())
		http.Redirect(w, r, "/", http.StatusTemporaryRedirect)
		return
	}
	// fmt.Println(token.AccessToken)
	resp, err := http.Get("https://www.googleapis.com/oauth2/v2/userinfo?access_token=" + token.AccessToken)
	if err != nil {
		fmt.Println("could not create request \n", err.Error())
		http.Redirect(w, r, "/", http.StatusTemporaryRedirect)
		return
	}

	defer resp.Body.Close()
	content, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("could not read response body \n", err.Error())
		http.Redirect(w, r, "/", http.StatusTemporaryRedirect)
		return
	}

	fmt.Fprintf(w, "Content: %s", content) //Menampilkan data yang didapat
}
```
11. Test dengan cara
```
go run main.go
```
