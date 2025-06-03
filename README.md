# abp-openiddict-rsa-sample
# ABP Framework: Paylaşımlı Hosting Ortamında PFX Sertifikası Olmadan OpenIddict Yayınlamak

## 🎯 Amaç

Bu repo, ABP Framework kullanılarak geliştirilen bir uygulamanın paylaşımlı hosting ortamında `.pfx` sertifikasına ihtiyaç duymadan OpenIddict ile token imzalama işlemini nasıl gerçekleştireceğini gösterir.

---

## ⚠️ Neden `.pfx` Dosyası Sorun Olur?

Paylaşımlı hosting ortamlarında:

- `.pfx` dosyasını okumak genellikle yasaktır,
- Uygulama başlatılamaz veya token üretimi başarısız olur.

---

## ✅ Çözüm: Kalıcı RSA Anahtarı ile Sertifikasız Token İmzalama

OpenIddict, `.pfx` yerine **kalıcı bir RSA özel anahtar dosyasını (PEM)** doğrudan kabul eder.

### 1. RSA Anahtarı Oluştur

openssl genpkey -algorithm RSA -out signing-key.pem -pkeyopt rsa_keygen_bits:2048

Oluşan dosyayı keys/ klasörüne yerleştirin.

### 2. Anahtarı Yükle (C#)

```csharp
using System.Security.Cryptography;

public static class RsaKeyLoader
{
    public static RSA LoadPrivateKey(string filePath)
    {
        var keyBytes = File.ReadAllBytes(filePath);
        var rsa = RSA.Create();
        rsa.ImportFromPem(System.Text.Encoding.UTF8.GetString(keyBytes));
        return rsa;
    }
}
```
ImportFromPem() .NET 5+ sürümlerinde desteklenir.

### 3. OpenIddict’e Entegre Et

MyProjectWebModule dosyasında:
```csharp
PreConfigure<OpenIddictServerBuilder>(builder =>
{
    var rsa = RsaKeyLoader.LoadPrivateKey(Path.Combine("keys", "signing-key.pem"));

    builder.AddSigningKey(new RsaSecurityKey(rsa)
    {
        KeyId = "my-signing-key"
    });
});
```

Güvenlik Tavsiyeleri
keys/signing-key.pem dosyasını .gitignore ile gizleyin:
```bash
/keys/*.pem
```
Üretim ortamında bu dosyayı kimseyle paylaşmayın.
