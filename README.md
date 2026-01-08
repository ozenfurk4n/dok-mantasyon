# Prolegal Bilgi Portalı / Stage App Dokümantasyonu  Bu belge, uygulamanın bileşenlerini, API uçlarını, veri modelini ve çalıştırma adımlarını tek yerde toplar. Kodun tamamı PHP/MySQL üzerinde çalışır, istemci tarafında Leaflet/Bootstrap/jQuery kullanılır ve tüm servisler `api.php?action=...` parametresiyle yönlendirilir.  ## İçindekiler - Genel Bakış ve Mimari - Teknoloji Yığını ve Bağımlılıklar - Dizinyapısı - Kurulum ve Çalıştırma - Kimlik Doğrulama ve Roller - Veritabanı ve Ana Tablolar - API Kullanım Kuralları - Genel API Uçları (`api.php`) - Talep Modülü API’si (`talep/api_requests.php`) - Ön Uç Modülleri - Loglama ve Audit - Operasyonel Notlar / TODO  ## Genel Bakış ve Mimari - Tek PHP uygulaması; oturum (session) tabanlı kimlik doğrulama, MySQL veritabanı. - Harita/dashbord ekranı: tapu/parsel veri sorgulama, firmaya atama, sorgu kuyruğu, KML/Excel dışa aktarma, imar bilgisi entegrasyonu. - Talep modülü: partner ve Prolegal ekiplerinin talep açma, parsel ekleme, statü takibi ve araştırma (research) süreçlerini yönetir. - API katmanı: `api.php` hem dashboard servislerini hem de talep API router’ını çalıştırır; tüm yanıtlar JSON’dur. - Audit katmanı: giriş denemeleri ve kritik aksiyonlar `audit.php` üzerinden loglanır.  ## Teknoloji Yığını ve Bağımlılıklar - Sunucu: PHP 8.x (mysqli, session, mbstring), MySQL 8+ (spatial fonksiyonlar kullanıldığı için `ST_GeomFromGeoJSON` vb. desteklenmeli). - Komut satırı bağımlılığı: `composer` (tek paket: `phpoffice/phpspreadsheet@^5.1` – Excel üretimi). - İstemci: Leaflet 1.7, Bootstrap 5, Font Awesome, jQuery 3.6, Google Fonts (icons). - Medya: `video.mp4` arka plan, `prolegal-logo.svg/png` logolar, KML dosyaları (`kml/`).  ## Dizinyapısı - `api.php` – Ana API gateway, dashboard/tapu/imar/şirket/sorgu operasyonları. - `talep/` – Talep UI ve API router (`talep/api_requests.php`), layout ve statik varlıklar. - `adminPanel.php` – Partner olmayan kullanıcılar için tapu/basvuru/sorgu kayıtlarını ID ile çekip düzenleyebilen hafif yönetim paneli; TKGM proxy testi ve yeni tapu oluşturma akışlarını kullanır. - `dinamik.php` – Harita/dataset filtreleri için dropdown verisi üreten bağımsız JSON servis (il/ilçe/mahalle + tür/hisse/sorgu kombinasyonlarına göre). - `tescil/` – Tescil teslimatı için redakte edilmiş API/JS kopyaları, PDF/text dosyaları ve teknoloji notları (canlı koddan bağımsız tutulur). - `assets/`, `tasarim/` – CSS/JS ve tasarım dosyaları. - `kml/` – KML verileri; `serve_kml` ve dışa aktarma uçları buradan servis edilir. - `il/`, `ilce/`, `mahalle/` – İdari alan veri kümeleri. - `vendor/` – Composer bağımlılıkları (PhpSpreadsheet). - `login.php`, `logout.php`, `index.php` – Giriş sayfası ve giriş sonrası yönlendirmeler. - `dashboard.php`, `dashboard_back.php` – Harita/dashbord ekranları ve arka plan JS/CSS dahil. - `audit.php` – Audit/login log fonksiyonları ve tablo üretimi. - Çeşitli yardımcılar: `export_excel.php`, `export_company_excel.php`, `webhook.php`, `SimpleXLSXGen.php` vb.  ## Kurulum ve Çalıştırma - Bu ortamda PHP, MySQL ve Composer bağımlılıkları zaten kurulu; ek kurulum yapmayın.   - Veritabanı varsayılan adı `tapu_backup_v2`; gerekirse `api.php` ve `audit.php` içindeki bağlantı bilgilerini düzenleyin.   - Sunucu hâlihazırda çalışıyorsa dokunmayın; yalnızca yerel test gerektiğinde PHP built‑in server ile kök dizini yayınlayabilirsiniz (`php -S localhost:8000`).   - Oturumlar için `session.save_path` yazılabilir olmalı; dosya izinlerini kontrol edin.   - Harici servis yok; tüm API çağrıları yerel MySQL’e gider.   - Üretim için: hata gösterimini kapatın, DB kimlik bilgilerini ortam değişkeni/secret yönetimine taşıyın, HTTPS zorunlu kılın.  ## Kimlik Doğrulama ve Roller - Giriş: `login.php` AJAX POST (`username`, `password`). Yanıt `success`, `role`, `username`. Başarılı girişte session değişkenleri (`loggedin`, `username`, `user_role`, `user_id`) set edilir. - Rate limit: IP başına 15 dakikada 35 deneme; aşıldığında bloklanır. - Roller: `partner`, `power_user`, `prolegal`, `admin`. Partner kullanıcıları doğrudan talep ekranına (`/talep/index.php`) yönlendirilir; diğer roller dashboard’a erişir. - Şifre kontrolü şu an düz metin eşleşmesiyle yapılır; `talep/api_requests.php` yeni kullanıcı oluştururken bcrypt kullanır. Canlıda tek tip (hash) kullanımı ve HTTPS önerilir. - Çıkış: `logout.php` session’ı temizler ve giriş sayfasına döner.  ## Veritabanı ve Ana Tablolar (özet) - Kimlik/Audit: `users`, `user_login_log`, `audit_log`. - Talep: `requests`, `request_parcels`, `request_status_history`, `request_revisions`, `research_requests`, `research_answers`, `subjects`, `firms`, `contract_flags`. - Tapu/Parsel: `tapu_maliye` (ana envanter + geometri), `sorgudurumu` (sorgu durumu), `basvurular` (başvurular), `popup_update_fail_log`, `parcel_actions` (kullanıcı-parsel aksiyonları), `tkgm_*` lookup tabloları. - Şirket/Portföy: `companies`, `parcel_company` (parsel-şirket eşlemesi), `firm_company_mapping`. - Kuyruk/Sorgu: `query_queue`, `queue_history`, `query_records`, `query_results`. - Destek: `request_status`, `research_status`, `mahalle`/`ilce`/`il` referansları.  ## API Kullanım Kuralları - Temel çağrı: `GET/POST /api.php?action=<ad>`; yanıtlar `Content-Type: application/json; charset=utf-8`. - Parametreler: çoğu uç `$_GET` veya `$_POST` okur; JSON body bekleyen uçlar `php://input` üzerinden parse edilir (özellikle talep API’sinde). - Hata kodları: 200 başarı, 400/404 validasyon/eksik parametre, 405 method hatası, 409 sürüm çakışması, 500 veritabanı hataları. - Oturum: Kimlik doğrulaması gereken uçlar aktif session ve rol bilgisine güvenir; token tabanlı auth yoktur. - Örnek cURL (talep ekleme):   ```bash   curl -X POST "http://localhost:8000/api.php?action=create_request" \        -H "Content-Type: application/json" \        -d '{"request_title":"Örnek Talep","request_type":"arastirma","subject_ids":[1,2]}'   ```  ## Genel API Uçları (`api.php`) Aşağıdaki başlıklar `api.php` içinde tanımlı uçları gruplayarak özetler. Çoğu uç GET ile okunur, POST ile güncellenir.  **Tapu / Parsel Veri** - `get_tapu_by_id`, `get_tapu_full`, `find_tapu_by_location`, `query_by_polygon`, `get_request_ils` – konum/ID ile parsel detayları ve GeoJSON geometri. - `create_tapu`, `update_tapu` – tapu_maliye kaydı oluşturma/güncelleme; geometri `polygon` alanı GeoJSON olarak gönderilir. - `update_sorgu_durumu`, `get_sorgu_durumu`, `update_basvuru_durumu`, `get_basvuru_durumu`, `get_property_status` – sorgu/başvuru/uygunluk durum yönetimi. - `update_imar_data`, `get_imar_bilgileri`, `get_imar_detay`, `imar_api_query`, `get_imar_ana_fonksiyonlari`, `get_imar_alt_fonksiyonlari`, `get_main_functions`, `get_sub_functions` – imar fonksiyonları ve detay sorguları. - `save_prolegal_data`, `save_prolegal_note`, `save_not_rating` – notlar ve puanlama.  **ATN / Normalizasyon** - `get_atn_suggestions`, `normalize_tapu_atn`, `rebuild_atn_lookup`, `reset_atn_lookup`, `generate_tapu_id` – ada/parsel/ATN normalizasyonu ve lookup yönetimi.  **Şirket / Portföy** - `get_companies`, `add_company`/`create_company`, `update_company`, `delete_company`, `get_companies_summary` – şirket CRUD (create_company yalnızca isimle hızlı ekleme). - `addPropertyToFirm`, `add_property_to_company`, `remove_property_from_company`, `removePropertyFromFirm`, `copy_company_properties`, `get_property_companies` – parsellerin şirketlere bağlanması veya çıkarılması. - `get_company_properties` – seçili şirketin parsellerini (geometri dahil) döner; sorgu/basvuru notlarını da içerir. - `create_company_from_firm`, `firm_company_mapping` – talep firmalarından şirket oluşturup eşleştirme.  **Sorgu Kuyruğu / İş Akışı** - `add_to_queue`, `batch_add_to_queue`, `add_to_query_queue`, `remove_from_queue`, `send_to_query`, `mark_as_suitable` – kuyruğa ekleme/çıkarma ve işaretleme. - `get_queue_list`, `get_query_queue`, `get_query_records`, `get_query_results`, `update_query_result`, `update_query_record`, `update_record_status`, `update_requests_status_by_parcels`, `cleanup_duplicate_sorgudurumu` – sorgu kuyruğu ve sonuç yönetimi. - `update_record_status` – tek çağrıda `sorgudurumu` (sorgulama_durumu), `basvurular` (basvuru_turu) ve `tapu_maliye`/`imar2` (durum/uygunluk) tablolarını günceller; boş değer gönderilirse sorgu durumu satırı silinir. - `set_sorgu_status` – `prolegal_id` listesi için `sorgudurumu` satırını tek seferde “Sırada” olarak yazar; JSON veya form-data kabul eder. - `get_all_records`, `get_query_history`, `get_dashboard_kpi`, `get_property_count` – dashboard rapor/kpi uçları.  **KML / Harici Servis** - `serve_kml`, `export_kml` – KML servis ve dışa aktarma. - `tkgm_parsel_proxy`, `tkgm_proxy_ip_test`, `tkgm_attribute_menu` – TKGM proxy ve test uçları. - `export_query_records` – sorgu kayıtlarını Excel/KML benzeri formatta dışa aktarma.  **Loglama / Destek** - `log_popup_update_fail` – popup güncelleme hatalarını kaydeder. - `get_parcel_actions`, `save_parcel_actions` – kullanıcı başına parsel üzerinde alınan aksiyonları (tapu sorgu, Google Earth, TKGM) loglar.  **Diğer** - `get_partners`, `get_firms`, `get_request_statuses`, `get_research_statuses`, `get_research_status_history`, `get_status_history` – lookup ve rapor uçları. - `update_multiple_property_status`, `update_property_status`, `add_property_to_company`, `remove_property_from_company` – toplu durum/güncelleme uçları.  ## api.php Detaylı Akış ve Endpoint Açıklamaları `api.php`, dashboard/tapu/imar/şirket/kuyruk operasyonlarının tamamını içerir ve talep router’ını (`talep/api_requests.php`) de içeri alır. Aşağıdaki başlıklar kod akışını ve tüm action’ları açıklar.  **Giriş ve Router** - Session başlatılır, `audit.php` yüklenir, MySQL bağlantısı açılır (utf8mb4). Bağlantı hatasında `addPropertyToFirm` isteğine JSON hata, diğerlerinde die() döner. - Talep action’ları `$talepActions` listesiyle `handleRequestsApi($conn)` fonksiyonuna devredilir (exit). Geri kalan action’lar bu dosyada işlenir. - `$action` GET/POST birleşiminden okunur; Content-Type her zaman JSON. Eşleşme olmazsa 405 döner; dosya sonunda bağlantı kapanır.  **Yardımcı Fonksiyonlar (özet)** - Biçimlendirme/normalizasyon: `formatYuzolcum`, `normalizeYuzolcumForStorage`, `toUpperTr`, `normalizeAtnValueForLookup`, `normalizedAtnSqlExpr`, `isNumericLike`, `normalizeDigitsToInt`. - Lookup ve tablo kontrolleri: `tableHasColumn`, `ensurePopupFailLogTable`, `ensure_parcel_actions_table`, `ensure_basvurular_table`. - ATN/ID işlemleri: `atnLookupExists`, `dropAtnLookup`, `rebuildAtnLookup`, `normalizeTapuAtn`, `generateUniqueTapuId`. - SQL yardımcıları: `stmtFetchFirstAssoc`, `stmtFetchAllAssoc`. Debug flag: `isDebugEnabled()`. - Parsel veri toplayıcılar: `fetchTasinmazDetayData`, `getParcelData` (tüm filtre kombinasyonları), geometri dönüştürücüleri `processPolygon`, `getFirstCoordinateFromWkt`, `convertWktToKml`. - Proxy yardımcıları: `buildBrightProxyCreds`, `maskProxyInfo`, `getProxyExitIp` (TKGM proxy testleri).  **Talep Statü Otomasyonu** - `maybe_advance_request_status` (POST): partner batch’i “araziler_onaylandı_sorgu” olan parsellerin tamamı `sorgulama_durumu='Sorgulandı'` olduğunda talep statüsünü 6→7 çeker, statü geçmişini loglar. - `maybe_set_request_status_sorguya_yollandi` (POST): aynı batch parselleri `Sorguda/Sorgulandı` ise statüyü 4→6 çeker; aksi durumda dokunmaz.  **ATN / ID / Normalizasyon Uçları** - `get_atn_suggestions` (GET): ada/parsel/ATN metnine göre öneri listesi. - `rebuild_atn_lookup` (GET): lookup tablosunu drop+create yapar. - `reset_atn_lookup` (GET): lookup temizler. - `normalize_tapu_atn` (GET/POST): ATN kolonlarını normalize eder, özet döner. - `generate_tapu_id` (GET): benzersiz tapu ID üretir (`generateUniqueTapuId`).  **TKGM / Proxy / KML** - `tkgm_attribute_menu` (GET): attribute menüsü (sabit JSON). - `tkgm_proxy_ip_test` (GET), `tkgm_parsel_proxy` (GET): BrightData proxy ile TKGM istekleri ve IP testi. - `serve_kml` (GET): `kml/` altındaki dosyayı servis eder. - `export_kml` (GET): WKT’yi KML’e çevirip döner. - `export_query_records` (GET): sorgu kayıtlarını tarih/durum filtreleriyle dışa aktarır.  **Tapu / Parsel CRUD ve Sorgu** - `get_tapu_by_id` (GET): tapu_maliye kaydını GeoJSON geometriyle döner. - `get_tapu_full` (GET): tapu + sorgu/başvuru/imar birleşik detay. - `find_tapu_by_location` (GET): lat/lng’e en yakın tapuyu getirir. - `query_by_polygon` (GET/POST): GeoJSON polygon alır, WKT kesişim sorgusu yapar, GeoJSON Feature listesi döner. - `create_tapu` (POST): yeni tapu kaydı (geometri dahil). - `update_tapu` (POST): tapu güncelleme, `row_version` kontrolü ve audit log içerir; GeoJSON doğrulaması yapar. - `get_property_status` / `update_property_status` / `update_multiple_property_status`: uygunluk durumu okuma/güncelleme (tekil/toplu). - `update_sorgu_durumu` / `get_sorgu_durumu`: sorgudurumu tablosu CRUD. - `update_basvuru_durumu` / `get_basvuru_durumu`: basvurular tablosu CRUD. - `save_prolegal_data`, `save_prolegal_note`, `save_not_rating`: prolegal not/puanlama alanları. - `get_main_functions`, `get_sub_functions`: popup fonksiyon listeleri.  **İmar Uçları** - `get_imar_bilgileri`, `get_imar_detay` (GET): parsel imar bilgileri ve detayları. - `get_imar_ana_fonksiyonlari`, `get_imar_alt_fonksiyonlari` (GET): lookup listeleri. - `imar_api_query` (GET): ada/parsel/id parametreleriyle imar sorgusu. - `update_imar_data` (POST): imar verisi güncelleme.  **Şirket / Portföy Yönetimi** - `get_companies` (GET): şirket listesi, `get_companies_summary` (GET): şirket + parsel sayısı. - `add_company`/`create_company`, `update_company`, `delete_company` (POST): şirket CRUD; `create_company` sadece isim alıp uniq kontrolü yapar. - `add_property_to_company`, `remove_property_from_company`, `get_property_companies` (POST/GET): parsel-şirket bağları. - `addPropertyToFirm`, `removePropertyFromFirm` (POST/GET): aynı işlemleri yapan eski isimli uçlar. - `copy_company_properties` (POST): şirketten şirkete parsel kopyası. - `get_company_properties` (GET): şirketin tüm parselleri; geometri, sorgu/basvuru/not dahil. - `create_company_from_firm` (POST): talep firmasından şirket oluşturur, `firm_company_mapping` kaydı yazar. - `get_property_count` (GET): şirket bazlı parsel sayısı.  **Sorgu Kuyruğu / İş Akışı ve Raporlar** - Kuyruğa ekleme/çıkarma: `add_to_queue`, `add_to_query_queue`, `batch_add_to_queue`, `remove_from_queue`, `send_to_query`, `mark_as_suitable`. - Sorgu/sonuçlar: `get_queue_list`, `get_query_queue`, `get_query_records`, `get_query_results`, `update_query_record`, `update_query_result`, `update_record_status`. - Temizlik/güncelleme: `update_requests_status_by_parcels` (talep statülerini parsel bazlı günceller), `cleanup_duplicate_sorgudurumu`. - Raporlama: `get_all_records`, `get_query_history`, `get_dashboard_kpi`, `get_request_ils`, `get_property_count`, `get_pending_query_requests`.  **Parsel Aksiyon ve Hata Logları** - `log_popup_update_fail` (POST): popup güncelleme hatalarını `popup_update_fail_log` tablosuna yazar. - `get_parcel_actions` / `save_parcel_actions` (GET/POST): kullanıcı bazlı aksiyon (TapuSorgu/Earth/TKGM) bayrakları; tablo otomatik oluşturulur. - `get_request_statuses`, `get_research_statuses`, `get_research_status_history` (GET): lookup/rapor uçları.  **Geo Dışa Aktarım ve Yardımcılar** - `query_by_polygon` ve `export_kml` dönüşleri `processPolygon` ile GeoJSON koordinat üretir. - `convertWktToKml` / `getFirstCoordinateFromWkt` KML export akışında kullanılır.  **Varsayılan Davranışlar** - Eşleşmeyen action’da 405 JSON hata döner; dosya sonunda `$conn->close()` çağrılır. - `api_back.php` eski/test amaçlı tutulur, ana akış `api.php` üzerindedir.  ## Talep Modülü API’si (`talep/api_requests.php`) Router, `api.php?action=<ad>` üzerinden çağrılır. Varsayılan yanıt formatı `{ success: bool, ... }`. Statüler `$REQUEST_STATUS` sözlüğüyle tanımlanır (1 Talep Alındı → 10 Park/İptal, 9 Arazi Seçimi Tamamlandı).  **Talep CRUD ve Statü** - `create_request`, `update_request`, `get_request`, `list_requests`, `delete_request`. - `update_request_status`, `update_request_priority`, `update_request_priorities`, `update_prolegal_priority`, `update_request_priorities` – statü ve öncelik yönetimi. - `get_active_requests`, `get_waiting_requests`, `get_unreviewed_requests`, `get_completed_requests`, `get_weekly_requests`, `get_filing_stage_requests`, `get_pending_query_requests` – listeleme/filtreleme raporları. - `update_request_priorities`, `update_request_priority` – toplu/tekil öncelik güncelleme. - `create_request_revision`, `get_request_revisions` – talep revizyon geçmişi.  **Parsel Yönetimi (Talep)** - `add_request_parcel`, `update_request_parcel`, `delete_request_parcel` – talebe parsel ekleme/güncelleme/silme. - `bulk_update_request_parcels` – `request_parcels` satırlarını JSON array ile toplu olarak status/comment/tkgm_link/parcel_documents alanları bazında günceller. - `list_request_parcels`, `list_request_parcels_by_parcel_ids`, `get_request_parcel`, `get_request_parcels_detailed` – talep-parsel listeleri. - `update_parcels_partner_batch` – partner uygunluğu/sorgu batch güncelleme. - `update_parcel_documents` – `multipart/form-data` ile tek dosya yükler, `/uploads` altına kaydeder ve ilgili talebin tüm parselleri için `parcel_documents` JSON alanını günceller (pdf/doc/docx/jpg/png, maks. 15 MB). - `maybe_advance_request_status`, `maybe_set_request_status_sorguya_yollandi` – parsel sorgu durumuna göre statüyü otomatik ilerletme.  **Kullanıcı, Firma, Konu** - `create_user`, `list_users`, `get_user`, `update_user` – kullanıcı CRUD (bcrypt ile kayıt). - `create_firm`, `list_firms`, `update_firm` – firma yönetimi. - `create_subject`, `list_subjects`, `update_subject` – konu/subject yönetimi. - `send_partner_email`,arth/TKGM tuşları `parcel_actions` tablosuna yazılır. - `talep/views/layout.php` – esnek sidebar + harita yerleşimi; talep oluşturma, parsel bağlama, statü geçişleri; partner ve prolegal rolleri için buton/görünürlük kontrolleri. - `adminPanel.php` – session korumalı, partner girişi kapalı; `tapu_maliye` kayıtlarını ID veya ada/parsel ile getirip düzenleyebilir, TKGM proxy testleri yapar, yeni tapu ID üretip kayıt açabilir, başvuru/sorgu detaylarını hızlı görüntüler.  ## Tescil Paketi / Redaksiyon - `tescil/README.txt` ve `teknoloji_ortam_bilgi.txt` tescil teslimi için kullanılan redakte edilmiş paketlerin içeriğini özetler. - Klasör; maskelenmiş `api_requests.php`, `form.js`, `list.js`, `imar_api.js`, PDF/TXT ekleri ve örnekler içerir; can# Prolegal Bilgi Portalı / Stage App Dokümantasyonu

Bu belge, uygulamanın bileşenlerini, API uçlarını, veri modelini ve çalıştırma adımlarını tek yerde toplar. Kodun tamamı PHP/MySQL üzerinde çalışır, istemci tarafında Leaflet/Bootstrap/jQuery kullanılır ve tüm servisler `api.php?action=...` parametresiyle yönlendirilir.

## İçindekiler
- Genel Bakış ve Mimari
- Teknoloji Yığını ve Bağımlılıklar
- Dizinyapısı
- Kurulum ve Çalıştırma
- Kimlik Doğrulama ve Roller
- Veritabanı ve Ana Tablolar
- API Kullanım Kuralları
- Genel API Uçları (`api.php`)
- Talep Modülü API’si (`talep/api_requests.php`)
- Ön Uç Modülleri
- Loglama ve Audit
- Operasyonel Notlar / TODO

## Genel Bakış ve Mimari
- Tek PHP uygulaması; oturum (session) tabanlı kimlik doğrulama, MySQL veritabanı.
- Harita/dashbord ekranı: tapu/parsel veri sorgulama, firmaya atama, sorgu kuyruğu, KML/Excel dışa aktarma, imar bilgisi entegrasyonu.
- Talep modülü: partner ve Prolegal ekiplerinin talep açma, parsel ekleme, statü takibi ve araştırma (research) süreçlerini yönetir.
- API katmanı: `api.php` hem dashboard servislerini hem de talep API router’ını çalıştırır; tüm yanıtlar JSON’dur.
- Audit katmanı: giriş denemeleri ve kritik aksiyonlar `audit.php` üzerinden loglanır.

## Teknoloji Yığını ve Bağımlılıklar
- Sunucu: PHP 8.x (mysqli, session, mbstring), MySQL 8+ (spatial fonksiyonlar kullanıldığı için `ST_GeomFromGeoJSON` vb. desteklenmeli).
- Komut satırı bağımlılığı: `composer` (tek paket: `phpoffice/phpspreadsheet@^5.1` – Excel üretimi).
- İstemci: Leaflet 1.7, Bootstrap 5, Font Awesome, jQuery 3.6, Google Fonts (icons).
- Medya: `video.mp4` arka plan, `prolegal-logo.svg/png` logolar, KML dosyaları (`kml/`).

## Dizinyapısı
- `api.php` – Ana API gateway, dashboard/tapu/imar/şirket/sorgu operasyonları.
- `talep/` – Talep UI ve API router (`talep/api_requests.php`), layout ve statik varlıklar.
- `adminPanel.php` – Partner olmayan kullanıcılar için tapu/basvuru/sorgu kayıtlarını ID ile çekip düzenleyebilen hafif yönetim paneli; TKGM proxy testi ve yeni tapu oluşturma akışlarını kullanır.
- `dinamik.php` – Harita/dataset filtreleri için dropdown verisi üreten bağımsız JSON servis (il/ilçe/mahalle + tür/hisse/sorgu kombinasyonlarına göre).
- `tescil/` – Tescil teslimatı için redakte edilmiş API/JS kopyaları, PDF/text dosyaları ve teknoloji notları (canlı koddan bağımsız tutulur).
- `assets/`, `tasarim/` – CSS/JS ve tasarım dosyaları.
- `kml/` – KML verileri; `serve_kml` ve dışa aktarma uçları buradan servis edilir.
- `il/`, `ilce/`, `mahalle/` – İdari alan veri kümeleri.
- `vendor/` – Composer bağımlılıkları (PhpSpreadsheet).
- `login.php`, `logout.php`, `index.php` – Giriş sayfası ve giriş sonrası yönlendirmeler.
- `dashboard.php`, `dashboard_back.php` – Harita/dashbord ekranları ve arka plan JS/CSS dahil.
- `audit.php` – Audit/login log fonksiyonları ve tablo üretimi.
- Çeşitli yardımcılar: `export_excel.php`, `export_company_excel.php`, `webhook.php`, `SimpleXLSXGen.php` vb.

## Kurulum ve Çalıştırma
- Bu ortamda PHP, MySQL ve Composer bağımlılıkları zaten kurulu; ek kurulum yapmayın.  
- Veritabanı varsayılan adı `tapu_backup_v2`; gerekirse `api.php` ve `audit.php` içindeki bağlantı bilgilerini düzenleyin.  
- Sunucu hâlihazırda çalışıyorsa dokunmayın; yalnızca yerel test gerektiğinde PHP built‑in server ile kök dizini yayınlayabilirsiniz (`php -S localhost:8000`).  
- Oturumlar için `session.save_path` yazılabilir olmalı; dosya izinlerini kontrol edin.  
- Harici servis yok; tüm API çağrıları yerel MySQL’e gider.  
- Üretim için: hata gösterimini kapatın, DB kimlik bilgilerini ortam değişkeni/secret yönetimine taşıyın, HTTPS zorunlu kılın.

## Kimlik Doğrulama ve Roller
- Giriş: `login.php` AJAX POST (`username`, `password`). Yanıt `success`, `role`, `username`. Başarılı girişte session değişkenleri (`loggedin`, `username`, `user_role`, `user_id`) set edilir.
- Rate limit: IP başına 15 dakikada 35 deneme; aşıldığında bloklanır.
- Roller: `partner`, `power_user`, `prolegal`, `admin`. Partner kullanıcıları doğrudan talep ekranına (`/talep/index.php`) yönlendirilir; diğer roller dashboard’a erişir.
- Şifre kontrolü şu an düz metin eşleşmesiyle yapılır; `talep/api_requests.php` yeni kullanıcı oluştururken bcrypt kullanır. Canlıda tek tip (hash) kullanımı ve HTTPS önerilir.
- Çıkış: `logout.php` session’ı temizler ve giriş sayfasına döner.

## Veritabanı ve Ana Tablolar (özet)
- Kimlik/Audit: `users`, `user_login_log`, `audit_log`.
- Talep: `requests`, `request_parcels`, `request_status_history`, `request_revisions`, `research_requests`, `research_answers`, `subjects`, `firms`, `contract_flags`.
- Tapu/Parsel: `tapu_maliye` (ana envanter + geometri), `sorgudurumu` (sorgu durumu), `basvurular` (başvurular), `popup_update_fail_log`, `parcel_actions` (kullanıcı-parsel aksiyonları), `tkgm_*` lookup tabloları.
- Şirket/Portföy: `companies`, `parcel_company` (parsel-şirket eşlemesi), `firm_company_mapping`.
- Kuyruk/Sorgu: `query_queue`, `queue_history`, `query_records`, `query_results`.
- Destek: `request_status`, `research_status`, `mahalle`/`ilce`/`il` referansları.

## API Kullanım Kuralları
- Temel çağrı: `GET/POST /api.php?action=<ad>`; yanıtlar `Content-Type: application/json; charset=utf-8`.
- Parametreler: çoğu uç `$_GET` veya `$_POST` okur; JSON body bekleyen uçlar `php://input` üzerinden parse edilir (özellikle talep API’sinde).
- Hata kodları: 200 başarı, 400/404 validasyon/eksik parametre, 405 method hatası, 409 sürüm çakışması, 500 veritabanı hataları.
- Oturum: Kimlik doğrulaması gereken uçlar aktif session ve rol bilgisine güvenir; token tabanlı auth yoktur.
- Örnek cURL (talep ekleme):
  ```bash
  curl -X POST "http://localhost:8000/api.php?action=create_request" \
       -H "Content-Type: application/json" \
       -d '{"request_title":"Örnek Talep","request_type":"arastirma","subject_ids":[1,2]}'
  ```

## Genel API Uçları (`api.php`)
Aşağıdaki başlıklar `api.php` içinde tanımlı uçları gruplayarak özetler. Çoğu uç GET ile okunur, POST ile güncellenir.

**Tapu / Parsel Veri**
- `get_tapu_by_id`, `get_tapu_full`, `find_tapu_by_location`, `query_by_polygon`, `get_request_ils` – konum/ID ile parsel detayları ve GeoJSON geometri.
- `create_tapu`, `update_tapu` – tapu_maliye kaydı oluşturma/güncelleme; geometri `polygon` alanı GeoJSON olarak gönderilir.
- `update_sorgu_durumu`, `get_sorgu_durumu`, `update_basvuru_durumu`, `get_basvuru_durumu`, `get_property_status` – sorgu/başvuru/uygunluk durum yönetimi.
- `update_imar_data`, `get_imar_bilgileri`, `get_imar_detay`, `imar_api_query`, `get_imar_ana_fonksiyonlari`, `get_imar_alt_fonksiyonlari`, `get_main_functions`, `get_sub_functions` – imar fonksiyonları ve detay sorguları.
- `save_prolegal_data`, `save_prolegal_note`, `save_not_rating` – notlar ve puanlama.

**ATN / Normalizasyon**
- `get_atn_suggestions`, `normalize_tapu_atn`, `rebuild_atn_lookup`, `reset_atn_lookup`, `generate_tapu_id` – ada/parsel/ATN normalizasyonu ve lookup yönetimi.

**Şirket / Portföy**
- `get_companies`, `add_company`/`create_company`, `update_company`, `delete_company`, `get_companies_summary` – şirket CRUD (create_company yalnızca isimle hızlı ekleme).
- `addPropertyToFirm`, `add_property_to_company`, `remove_property_from_company`, `removePropertyFromFirm`, `copy_company_properties`, `get_property_companies` – parsellerin şirketlere bağlanması veya çıkarılması.
- `get_company_properties` – seçili şirketin parsellerini (geometri dahil) döner; sorgu/basvuru notlarını da içerir.
- `create_company_from_firm`, `firm_company_mapping` – talep firmalarından şirket oluşturup eşleştirme.

**Sorgu Kuyruğu / İş Akışı**
- `add_to_queue`, `batch_add_to_queue`, `add_to_query_queue`, `remove_from_queue`, `send_to_query`, `mark_as_suitable` – kuyruğa ekleme/çıkarma ve işaretleme.
- `get_queue_list`, `get_query_queue`, `get_query_records`, `get_query_results`, `update_query_result`, `update_query_record`, `update_record_status`, `update_requests_status_by_parcels`, `cleanup_duplicate_sorgudurumu` – sorgu kuyruğu ve sonuç yönetimi.
- `update_record_status` – tek çağrıda `sorgudurumu` (sorgulama_durumu), `basvurular` (basvuru_turu) ve `tapu_maliye`/`imar2` (durum/uygunluk) tablolarını günceller; boş değer gönderilirse sorgu durumu satırı silinir.
- `set_sorgu_status` – `prolegal_id` listesi için `sorgudurumu` satırını tek seferde “Sırada” olarak yazar; JSON veya form-data kabul eder.
- `get_all_records`, `get_query_history`, `get_dashboard_kpi`, `get_property_count` – dashboard rapor/kpi uçları.

**KML / Harici Servis**
- `serve_kml`, `export_kml` – KML servis ve dışa aktarma.
- `tkgm_parsel_proxy`, `tkgm_proxy_ip_test`, `tkgm_attribute_menu` – TKGM proxy ve test uçları.
- `export_query_records` – sorgu kayıtlarını Excel/KML benzeri formatta dışa aktarma.

**Loglama / Destek**
- `log_popup_update_fail` – popup güncelleme hatalarını kaydeder.
- `get_parcel_actions`, `save_parcel_actions` – kullanıcı başına parsel üzerinde alınan aksiyonları (tapu sorgu, Google Earth, TKGM) loglar.

**Diğer**
- `get_partners`, `get_firms`, `get_request_statuses`, `get_research_statuses`, `get_research_status_history`, `get_status_history` – lookup ve rapor uçları.
- `update_multiple_property_status`, `update_property_status`, `add_property_to_company`, `remove_property_from_company` – toplu durum/güncelleme uçları.

## api.php Detaylı Akış ve Endpoint Açıklamaları
`api.php`, dashboard/tapu/imar/şirket/kuyruk operasyonlarının tamamını içerir ve talep router’ını (`talep/api_requests.php`) de içeri alır. Aşağıdaki başlıklar kod akışını ve tüm action’ları açıklar.

**Giriş ve Router**
- Session başlatılır, `audit.php` yüklenir, MySQL bağlantısı açılır (utf8mb4). Bağlantı hatasında `addPropertyToFirm` isteğine JSON hata, diğerlerinde die() döner.
- Talep action’ları `$talepActions` listesiyle `handleRequestsApi($conn)` fonksiyonuna devredilir (exit). Geri kalan action’lar bu dosyada işlenir.
- `$action` GET/POST birleşiminden okunur; Content-Type her zaman JSON. Eşleşme olmazsa 405 döner; dosya sonunda bağlantı kapanır.

**Yardımcı Fonksiyonlar (özet)**
- Biçimlendirme/normalizasyon: `formatYuzolcum`, `normalizeYuzolcumForStorage`, `toUpperTr`, `normalizeAtnValueForLookup`, `normalizedAtnSqlExpr`, `isNumericLike`, `normalizeDigitsToInt`.
- Lookup ve tablo kontrolleri: `tableHasColumn`, `ensurePopupFailLogTable`, `ensure_parcel_actions_table`, `ensure_basvurular_table`.
- ATN/ID işlemleri: `atnLookupExists`, `dropAtnLookup`, `rebuildAtnLookup`, `normalizeTapuAtn`, `generateUniqueTapuId`.
- SQL yardımcıları: `stmtFetchFirstAssoc`, `stmtFetchAllAssoc`. Debug flag: `isDebugEnabled()`.
- Parsel veri toplayıcılar: `fetchTasinmazDetayData`, `getParcelData` (tüm filtre kombinasyonları), geometri dönüştürücüleri `processPolygon`, `getFirstCoordinateFromWkt`, `convertWktToKml`.
- Proxy yardımcıları: `buildBrightProxyCreds`, `maskProxyInfo`, `getProxyExitIp` (TKGM proxy testleri).

**Talep Statü Otomasyonu**
- `maybe_advance_request_status` (POST): partner batch’i “araziler_onaylandı_sorgu” olan parsellerin tamamı `sorgulama_durumu='Sorgulandı'` olduğunda talep statüsünü 6→7 çeker, statü geçmişini loglar.
- `maybe_set_request_status_sorguya_yollandi` (POST): aynı batch parselleri `Sorguda/Sorgulandı` ise statüyü 4→6 çeker; aksi durumda dokunmaz.

**ATN / ID / Normalizasyon Uçları**
- `get_atn_suggestions` (GET): ada/parsel/ATN metnine göre öneri listesi.
- `rebuild_atn_lookup` (GET): lookup tablosunu drop+create yapar.
- `reset_atn_lookup` (GET): lookup temizler.
- `normalize_tapu_atn` (GET/POST): ATN kolonlarını normalize eder, özet döner.
- `generate_tapu_id` (GET): benzersiz tapu ID üretir (`generateUniqueTapuId`).

**TKGM / Proxy / KML**
- `tkgm_attribute_menu` (GET): attribute menüsü (sabit JSON).
- `tkgm_proxy_ip_test` (GET), `tkgm_parsel_proxy` (GET): BrightData proxy ile TKGM istekleri ve IP testi.
- `serve_kml` (GET): `kml/` altındaki dosyayı servis eder.
- `export_kml` (GET): WKT’yi KML’e çevirip döner.
- `export_query_records` (GET): sorgu kayıtlarını tarih/durum filtreleriyle dışa aktarır.

**Tapu / Parsel CRUD ve Sorgu**
- `get_tapu_by_id` (GET): tapu_maliye kaydını GeoJSON geometriyle döner.
- `get_tapu_full` (GET): tapu + sorgu/başvuru/imar birleşik detay.
- `find_tapu_by_location` (GET): lat/lng’e en yakın tapuyu getirir.
- `query_by_polygon` (GET/POST): GeoJSON polygon alır, WKT kesişim sorgusu yapar, GeoJSON Feature listesi döner.
- `create_tapu` (POST): yeni tapu kaydı (geometri dahil).
- `update_tapu` (POST): tapu güncelleme, `row_version` kontrolü ve audit log içerir; GeoJSON doğrulaması yapar.
- `get_property_status` / `update_property_status` / `update_multiple_property_status`: uygunluk durumu okuma/güncelleme (tekil/toplu).
- `update_sorgu_durumu` / `get_sorgu_durumu`: sorgudurumu tablosu CRUD.
- `update_basvuru_durumu` / `get_basvuru_durumu`: basvurular tablosu CRUD.
- `save_prolegal_data`, `save_prolegal_note`, `save_not_rating`: prolegal not/puanlama alanları.
- `get_main_functions`, `get_sub_functions`: popup fonksiyon listeleri.

**İmar Uçları**
- `get_imar_bilgileri`, `get_imar_detay` (GET): parsel imar bilgileri ve detayları.
- `get_imar_ana_fonksiyonlari`, `get_imar_alt_fonksiyonlari` (GET): lookup listeleri.
- `imar_api_query` (GET): ada/parsel/id parametreleriyle imar sorgusu.
- `update_imar_data` (POST): imar verisi güncelleme.

**Şirket / Portföy Yönetimi**
- `get_companies` (GET): şirket listesi, `get_companies_summary` (GET): şirket + parsel sayısı.
- `add_company`/`create_company`, `update_company`, `delete_company` (POST): şirket CRUD; `create_company` sadece isim alıp uniq kontrolü yapar.
- `add_property_to_company`, `remove_property_from_company`, `get_property_companies` (POST/GET): parsel-şirket bağları.
- `addPropertyToFirm`, `removePropertyFromFirm` (POST/GET): aynı işlemleri yapan eski isimli uçlar.
- `copy_company_properties` (POST): şirketten şirkete parsel kopyası.
- `get_company_properties` (GET): şirketin tüm parselleri; geometri, sorgu/basvuru/not dahil.
- `create_company_from_firm` (POST): talep firmasından şirket oluşturur, `firm_company_mapping` kaydı yazar.
- `get_property_count` (GET): şirket bazlı parsel sayısı.

**Sorgu Kuyruğu / İş Akışı ve Raporlar**
- Kuyruğa ekleme/çıkarma: `add_to_queue`, `add_to_query_queue`, `batch_add_to_queue`, `remove_from_queue`, `send_to_query`, `mark_as_suitable`.
- Sorgu/sonuçlar: `get_queue_list`, `get_query_queue`, `get_query_records`, `get_query_results`, `update_query_record`, `update_query_result`, `update_record_status`.
- Temizlik/güncelleme: `update_requests_status_by_parcels` (talep statülerini parsel bazlı günceller), `cleanup_duplicate_sorgudurumu`.
- Raporlama: `get_all_records`, `get_query_history`, `get_dashboard_kpi`, `get_request_ils`, `get_property_count`, `get_pending_query_requests`.

**Parsel Aksiyon ve Hata Logları**
- `log_popup_update_fail` (POST): popup güncelleme hatalarını `popup_update_fail_log` tablosuna yazar.
- `get_parcel_actions` / `save_parcel_actions` (GET/POST): kullanıcı bazlı aksiyon (TapuSorgu/Earth/TKGM) bayrakları; tablo otomatik oluşturulur.
- `get_request_statuses`, `get_research_statuses`, `get_research_status_history` (GET): lookup/rapor uçları.

**Geo Dışa Aktarım ve Yardımcılar**
- `query_by_polygon` ve `export_kml` dönüşleri `processPolygon` ile GeoJSON koordinat üretir.
- `convertWktToKml` / `getFirstCoordinateFromWkt` KML export akışında kullanılır.

**Varsayılan Davranışlar**
- Eşleşmeyen action’da 405 JSON hata döner; dosya sonunda `$conn->close()` çağrılır.
- `api_back.php` eski/test amaçlı tutulur, ana akış `api.php` üzerindedir.

## Talep Modülü API’si (`talep/api_requests.php`)
Router, `api.php?action=<ad>` üzerinden çağrılır. Varsayılan yanıt formatı `{ success: bool, ... }`. Statüler `$REQUEST_STATUS` sözlüğüyle tanımlanır (1 Talep Alındı → 10 Park/İptal, 9 Arazi Seçimi Tamamlandı).

**Talep CRUD ve Statü**
- `create_request`, `update_request`, `get_request`, `list_requests`, `delete_request`.
- `update_request_status`, `update_request_priority`, `update_request_priorities`, `update_prolegal_priority`, `update_request_priorities` – statü ve öncelik yönetimi.
- `get_active_requests`, `get_waiting_requests`, `get_unreviewed_requests`, `get_completed_requests`, `get_weekly_requests`, `get_filing_stage_requests`, `get_pending_query_requests` – listeleme/filtreleme raporları.
- `update_request_priorities`, `update_request_priority` – toplu/tekil öncelik güncelleme.
- `create_request_revision`, `get_request_revisions` – talep revizyon geçmişi.

**Parsel Yönetimi (Talep)**
- `add_request_parcel`, `update_request_parcel`, `delete_request_parcel` – talebe parsel ekleme/güncelleme/silme.
- `bulk_update_request_parcels` – `request_parcels` satırlarını JSON array ile toplu olarak status/comment/tkgm_link/parcel_documents alanları bazında günceller.
- `list_request_parcels`, `list_request_parcels_by_parcel_ids`, `get_request_parcel`, `get_request_parcels_detailed` – talep-parsel listeleri.
- `update_parcels_partner_batch` – partner uygunluğu/sorgu batch güncelleme.
- `update_parcel_documents` – `multipart/form-data` ile tek dosya yükler, `/uploads` altına kaydeder ve ilgili talebin tüm parselleri için `parcel_documents` JSON alanını günceller (pdf/doc/docx/jpg/png, maks. 15 MB).
- `maybe_advance_request_status`, `maybe_set_request_status_sorguya_yollandi` – parsel sorgu durumuna göre statüyü otomatik ilerletme.

**Kullanıcı, Firma, Konu**
- `create_user`, `list_users`, `get_user`, `update_user` – kullanıcı CRUD (bcrypt ile kayıt).
- `create_firm`, `list_firms`, `update_firm` – firma yönetimi.
- `create_subject`, `list_subjects`, `update_subject` – konu/subject yönetimi.
- `send_partner_email` – partner bilgilendirme e-postası tetikleme; HTML içerik PHP `mail()` ile gönderilir, il/ilçe adları JSON dosyalarından okunur, sunucuda MTA yapılandırması gereklidir.
- `get_request_ils` – talep filtreleme için il listesi (dashboard entegrasyonu).

**Araştırma / Research**
- `create_research_request`, `update_research_request`, `get_research_request`, `list_research_requests`, `update_research_request_status`, `answer_research_request`, `mark_research_answer_read` – araştırma talepleri iş akışı.
- `create_research_answer`, `list_research_answers`, `get_research_answer`, `update_research_answer` – araştırma cevapları.
- `get_research_status_history` – statü geçmişi.

**Sözleşme/Flag**
- `upsert_contract_flags`, `get_contract_flags` – sözleşme/flag bilgisi saklama ve okuma.

## Dinamik Filtre Servisi (`dinamik.php`)
- GET tabanlı; `type` parametresi `tasinmazDetay` (AnaTasinmazNitelik_1 listesini döner), `turSecimi` (uygun/diğer/arazi dışı kategorileri) veya `araziBuyuklugu` (sabit büyüklük aralıkları) değerlerini alır.
- Filtreler: `il`, `ilce` (virgülle çoklu), `mahalle`, `turSecimi` (0=uygun arazi, 1=diğer arazi, 2=arazi dışı), `hisseDurumu` (0 tam/0-1/1 üzeri, 1 paylı), `sorgulamaDurumu` (Durum kolonu).
- JSON yanıt döner; yalnızca polygonu ve nitelik bilgisi dolu `tapu_maliye` kayıtları listelenir.

## İmar API Konfigürasyonu ve Örnekler
- `imar_api_query` çağrısı, opsiyonel `imar_config.php` içindeki `api_url`/`headers` veya ortam değişkeni `IMAR_API_URL` üzerinden uzak imar servisine cURL yapar (varsayılan: `https://imar-app-rho.vercel.app/api/query`).
- `imar_config.php` yoksa oluşturulabilir; Authorization gibi ek header'lar `headers` dizisine eklenir.
- `php_client_example.php` dosyası Vercel imar API’sine örnek POST isteği (il/ilçe/mahalle/ada/parsel) ve dönen `plan_function`/`kaks`/`taks` değerlerinin nasıl işlendiğini gösterir.

## Ön Uç Modülleri
- `index.php` – giriş ekranı; video arka plan ve üç buton: Harita Sistemi (`dashboard.php`), Teşvik Asistanı (dış link), Talep Formu (`talep/index.php`).
- `dashboard.php` / `dashboard_back.php`
  - Leaflet tabanlı harita, parsel listesi, filtreler (il/ilçe/mahalle, ada/parsel, nitelik, alan, sorgu/başvuru/uygunluk durumu, tarih aralığı).
  - Şirket yönetimi: şirket ekleme/silme, parsel-şirket eşleme, şirkete göre listeleme.
  - Sorgu kuyruğu: parselleri sıraya ekleme/çıkarma, “Sorguda/Sırada” gibi durum takibi, batch işlemler.
  - İmar entegrasyonu: imar fonksiyonları ve detaylarını gösterme/güncelleme.
  - Export: Excel/KML çıktıları, sorgu kayıtları exportu, şirket bazlı Excel.
  - Popup aksiyon loglama: TapuSorgu/GoogleEarth/TKGM tuşları `parcel_actions` tablosuna yazılır.
- `talep/views/layout.php` – esnek sidebar + harita yerleşimi; talep oluşturma, parsel bağlama, statü geçişleri; partner ve prolegal rolleri için buton/görünürlük kontrolleri.
- `adminPanel.php` – session korumalı, partner girişi kapalı; `tapu_maliye` kayıtlarını ID veya ada/parsel ile getirip düzenleyebilir, TKGM proxy testleri yapar, yeni tapu ID üretip kayıt açabilir, başvuru/sorgu detaylarını hızlı görüntüler.

## Tescil Paketi / Redaksiyon
- `tescil/README.txt` ve `teknoloji_ortam_bilgi.txt` tescil teslimi için kullanılan redakte edilmiş paketlerin içeriğini özetler.
- Klasör; maskelenmiş `api_requests.php`, `form.js`, `list.js`, `imar_api.js`, PDF/TXT ekleri ve örnekler içerir; canlı uygulama tarafından kullanılmaz, arşiv/tescil amaçlıdır.

## Loglama ve Audit
- `audit.php` otomatik olarak `user_login_log` ve `audit_log` tablolarını oluşturur.
  - `log_login_attempt($username, $success, $reason)` – giriş denemesi kaydı (IP, UA).
  - `audit_log($action, $meta, $resource_type, $resource_id)` – genel aksiyon logu (session’daki kullanıcı adı öncelikli).
- Ek loglar: `api_log.txt` (eski `api_back.php`), `deploy/logs/deploy.log`, popup/parsel aksiyon logları, sorgu kuyruğu geçmişi.

## Deploy / Webhook
- `webhook.php` Windows/XAMPP stage ortamında GitHub `push` (branch `main`) için HMAC doğrulamalı in-place deploy yapar; `deploy.lock` ile kilit, `deploy/logs/deploy.log` ile kayıt tutar.
- Konfig: `repo_remote`, `workdir` (`C:\xampp\htdocs\stage` varsayılır), `git_bin`/`php_bin` yolları ve `secret` değeri güncel olmalıdır; `last_sha` ile idempotent çalışır.

## Ortam Değişkenleri ve Debug
- TKGM proxy: `BRIGHT_PROXY_HOST`/`PORT`/`USERPWD` (veya `BRIGHTDATA_HOST`/`PORT`/`USERNAME`/`PASSWORD`) tanımlanabilir; `BRIGHT_PROXY_ROTATE=1` her istek için yeni session açar.
- İmar API: `IMAR_API_URL` env veya `imar_config.php` içeriği ile override edilir.
- Debug: `APP_DEBUG=1` veya istek parametresi `debug=1` detaylı loglamayı açar; üretimde kapalı tutun.
- Dosya yüklemeleri: `update_parcel_documents` için `/uploads` dizini yazılabilir olmalı, PHP upload limitleri 15 MB üzeri izin vermemelidir; `session.save_path` yazılabilirliği kurulum sırasında kontrol edilmiş olmalı.

## Operasyonel Notlar / TODO
- DB bağlantı bilgilerini ortam değişkenine taşıyın; PHP dosyalarındaki sabit değerler (root/localhost) üretim için güncellenmeli.
- Girişte hash kontrolü ile `password_verify` kullanımı güvenlik için önerilir (kayıt tarafı bcrypt destekli).
- Uzun API listesini yönetilebilir kılmak için Swagger/OpenAPI dökümanı üretmek iyi olacaktır; `action` parametresi yerine RESTful URI’lar planlanabilir.
- MySQL spatial fonksiyonları kullanıldığı için sunucu tarafında `innodb` ve `utf8mb4_turkish_ci` desteği aktif olmalı; `tapu_maliye.polygon` alanı için SRID kontrolü eklenebilir.
- Büyük sorgular (`get_all_records`, `query_by_polygon`) için indeks/limit ve sayfalama uygulanması önerilir; şu an limitler çoğunlukla 500+.
- Prod izleme: `error_log` çıktıları gözden geçirilmeli, debug logları (`🔍`) kapatılmalı.
lı uygulama tarafından kullanılmaz, arşiv/tescil amaçlıdır.  ## Loglama ve Audit - `audit.php` otomatik olara[DOCUMENTATION.md](https://github.com/user-attachments/files/24498498/DOCUMENTATION.md)
k `user_login_log` ve `audit_log` tablolarını oluşturur.   - `log_login_attempt($username, $success, $reason)` – giriş denemesi kaydı (IP, UA).   - `audit_log($action, $meta, $resource_type, $resource_id)` – genel aksiyon logu (session’daki kullanıcı adı öncelikli). - Ek loglar: `api_log.txt` (eski `api_back.php`), `deploy/logs/deploy.log`, popup/parsel aksiyon logları, sorgu kuyruğu geçmişi.  ## Deploy / Webhook - `webhook.php` Windows/XAMPP stage ortamında GitHub `push` (branch `main`) için HMAC doğrulamalı in-place deploy yapar; `deploy.lock` ile kilit, `deploy/logs/deploy.log` ile kayıt tutar. - Konfig: `repo_remote`, `workdir` (`C:\xampp\htdocs\stage` varsayılır), `git_bin`/`php_bin` yolları ve `secret` değeri güncel olmalıdır; `last_sha` ile idempotent çalışır.  ## Ortam Değişkenleri ve Debug - TKGM proxy: `BRIGHT_PROXY_HOST`/`PORT`/`USERPWD` (veya `BRIGHTDATA_HOST`/`PORT`/`USERNAME`/`PASSWORD`) tanımlanabilir; `BRIGHT_PROXY_ROTATE=1` her istek için yeni session açar. - İmar API: `IMAR_API_URL` env veya `imar_config.php` içeriği ile override edilir. - Debug: `APP_DEBUG=1` veya istek parametresi `debug=1` detaylı loglamayı açar; üretimde kapalı tutun. - Dosya yüklemeleri: `update_parcel_documents` için `/uploads` dizini yazılabilir olmalı, PHP upload limitleri 15 MB üzeri izin vermemelidir; `session.save_path` yazılabilirliği kurulum sırasında kontrol edilmiş olmalı.  ## Operasyonel Notlar / TODO - DB bağlantı bilgilerini ortam değişkenine taşıyın; PHP dosyalarındaki sabit değerler (root/localhost) üretim için güncellenmeli. - Girişte hash kontrolü ile `password_verify` kullanımı güvenlik için önerilir (kayıt tarafı bcrypt destekli). - Uzun API listesini yönetilebilir kılmak için Swagger/OpenAPI dökümanı üretmek iyi olacaktır; `action` parametresi yerine RESTful URI’lar planlanabilir. - MySQL spatial fonksiyonları kullanıldığı için sunucu tarafında `innodb` ve `utf8mb4_turkish_ci` desteği aktif olmalı; `tapu_maliye.polygon` alanı için SRID kontrolü eklenebilir. - Büyük sorgular (`get_all_records`, `query_by_polygon`) için indeks/limit ve sayfalama uygulanması önerilir; şu an limitler çoğunlukla 500+. - Prod izleme: `error_log` çıktıları gözden geçirilmeli, debug logları (`🔍`) kapatılmalı.
