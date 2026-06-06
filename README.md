# modul-auth

Autentikasi untuk bahasa [Tenun](https://github.com/TenunLang/Tenun): hashing sandi dengan PBKDF2-HMAC-SHA256 dan token JWT HS256. Ditulis sepenuhnya dengan Tenun.

## Pasang

```
tenun add auth
```

## Pakai

```tenun
impor "auth";

// Saat daftar: simpan hash, bukan sandi asli.
biar simpan: teks = auth_hash(sandi_dari_form);

// Saat login: verifikasi.
kalau auth_verifikasi(sandi_dari_form, simpan) {
    biar token: teks = jwt_buat("{\"sub\":\"42\"}", "kunci-rahasia-server");
    headerKan("Set-Cookie", "token=" + token + "; HttpOnly; Path=/");
}
```

## Fungsi

### Sandi

- `auth_hash(sandi: teks): teks`
  Hasilkan hash aman. Format: `pbkdf2$<iter>$<salt_b64>$<hash_b64>`. Salt acak otomatis.
- `auth_verifikasi(sandi: teks, simpan: teks): bool`
  Verifikasi sandi terhadap nilai dari `auth_hash`. Perbandingan waktu konstan.

### JWT (HS256)

- `jwt_buat(payload: teks, rahasia: teks): teks` — payload JSON, kembalikan token.
- `jwt_verifikasi(token: teks, rahasia: teks): bool` — cek tanda tangan (waktu konstan).
- `jwt_payload(token: teks): teks` — ambil payload JSON. **Verifikasi dulu** sebelum percaya isinya.

### Token sesi (JWT + kedaluwarsa) & header

```tenun
biar token: teks = auth_jwt("42", "kunci-rahasia", 3600);  // berlaku 1 jam
headerKan("Set-Cookie", "token=" + token + "; HttpOnly; Path=/");

// middleware: ambil & cek token Bearer
biar t: teks = auth_bearer();
kalau auth_cek(t, "kunci-rahasia") {
    cetak("pengguna " + auth_subjek(t));
} lain {
    statusKan(401);
}
```

- `auth_jwt(sub, rahasia, ttl_detik): teks` — JWT dengan klaim `sub`/`iat`/`exp`.
- `auth_cek(token, rahasia): bool` — tanda tangan valid **dan** belum kedaluwarsa.
- `auth_subjek(token): teks`, `auth_klaim(token, kunci): teks` — baca klaim.
- `auth_bearer(): teks` — token dari header `Authorization: Bearer <token>`.
- `auth_token_acak(nbyte): teks` — token acak base64url (API key / reset / verifikasi email).

## Keamanan

- PBKDF2 120.000 iterasi (HMAC-SHA256). Salt 16 byte acak per sandi.
- Perbandingan hash/tanda tangan dilakukan dalam waktu konstan untuk mencegah timing attack.
- JWT memakai HMAC-SHA256; jaga kerahasiaan `rahasia`. Token tidak terenkripsi — jangan taruh data sensitif di payload.

## Struktur

```
modul-auth/
  tenun.json
  src/
    auth.tenun    hashing sandi (PBKDF2) + token sesi JWT, Bearer, token acak
    jwt.tenun     JWT HS256 (encode/decode/verify)
  contoh/contoh.tenun
```

## Lisensi

MIT.
