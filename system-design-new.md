# GoMad Unified Design System v1.0
## Revised Business Architecture & Multi-Experience Design Foundation

**Document Type:** Product Design & UI/UX Architecture  
**Version:** 1.0 — Revised Foundation  
**Date:** 2 November 2026  
**Status:** Foundational Design Document  
**Scope:** GoMad Core, MadWarung, MadTrans, Public

---

# 1. Executive Summary

GoMad merupakan platform bisnis digital berbasis vertical business ecosystem yang menghubungkan berbagai pelaku usaha melalui satu platform utama.

GoMad Core berfungsi sebagai pusat pengelolaan ekosistem, sedangkan MadWarung dan MadTrans beroperasi sebagai vertical business dengan model intermediary atau aggregator yang berbeda.

GoMad Unified Design System dikembangkan untuk memastikan seluruh produk memiliki identitas visual yang konsisten, tetapi tetap memberikan pengalaman antarmuka yang sesuai dengan karakteristik bisnis, aktivitas pengguna, intensitas transaksi, dan kanal akses masing-masing.

Design System ini tidak menggunakan pendekatan satu tampilan untuk semua produk.

Sebaliknya, GoMad menerapkan konsep:

**One Business Ecosystem, Multiple Usage Experiences.**

Satu ekosistem bisnis, satu identitas merek, dan beberapa pendekatan pengalaman pengguna.

# 2. Business Architecture

## 2.1 GoMad Core

GoMad Core merupakan platform utama yang membawahi dan mengelola keseluruhan ekosistem GoMad.

Fungsi utamanya meliputi:
- Central business management.
- User and role management.
- Vertical business management.
- Transaction monitoring.
- Financial and operational monitoring.
- Platform configuration.
- Business analytics and reporting.
- System governance.

GoMad Core memiliki dua kategori pengguna:

| Role | Tanggung Jawab |
|---|---|
| Admin | Pengelolaan keseluruhan platform, konfigurasi, akses, kebijakan, dan pengawasan bisnis |
| Ops | Pengelolaan aktivitas operasional, monitoring transaksi, dukungan pengguna, dan pelaksanaan proses bisnis |

**Access Channel:** Website only.

## 2.2 MadWarung

MadWarung merupakan vertical business GoMad yang berfungsi sebagai aggregator/intermediary antara grosir dan retail.

Model bisnis saat ini adalah B2B.

**Business Relationship:**

Grosir ↔ Retail

MadWarung menyediakan infrastruktur digital untuk menghubungkan kebutuhan pengadaan retail dengan penawaran dan ketersediaan produk dari grosir.

Karakteristik bisnis:
- High transaction frequency.
- Repetitive purchasing activities.
- Frequent order processing.
- Product catalog management.
- Wholesale and retail transaction management.
- Inventory and fulfillment coordination.

### Current Business Model: B2B

Fokus utama:
- Grosir menyediakan produk.
- Retail melakukan pengadaan.
- MadWarung menjadi intermediary.
- Transaksi berlangsung antara pelaku usaha.

### Future Business Model: B2B2C

MadWarung memiliki potensi ekspansi menuju model B2B2C.

Ekspansi tersebut memungkinkan konsumen akhir mengakses produk melalui jaringan retail yang telah terhubung dalam ekosistem MadWarung.

Namun, B2B2C merupakan pengembangan masa depan dan bukan bagian dari scope implementasi model bisnis saat ini.

### MadWarung Roles

| Role | Fungsi |
|---|---|
| Grosir | Mengelola produk, harga, stok, dan pesanan grosir |
| Retail | Mengelola pengadaan, katalog, transaksi, dan aktivitas usaha retail |
| Penjaga Warung Retail | Menjalankan aktivitas transaksi dan pelayanan di toko retail |

### Access Channel

| Role | Website | Mobile App |
|---|---|---|
| Grosir | Yes | No |
| Retail Owner / Management | Yes | No |
| Penjaga Warung Retail | No | Yes |

**Prinsip:** Seluruh aktivitas manajemen dan operasional bisnis MadWarung dilakukan melalui website. Aplikasi mobile hanya diperuntukkan bagi Penjaga Warung Retail untuk menjalankan aktivitas operasional di toko.

## 2.3 MadTrans

MadTrans merupakan vertical business GoMad yang berfungsi sebagai intermediary antara pemilik usaha transportasi dan pelanggan secara langsung.

MadTrans menghubungkan operator travel dan rental dengan penumpang yang membutuhkan layanan perjalanan.

**Business Relationship:**

Owner Travel/Rental ↔ Customer

MadTrans tidak harus memiliki armada atau menyediakan supply transportasi sendiri. Supply berasal dari operator yang bergabung dalam ekosistem.

Cakupan layanan:
- Intercity travel.
- Door-to-door travel.
- Car charter.
- Vehicle rental.
- Travel and rental booking.
- Schedule management.
- Driver management.
- Passenger management.
- Payment and wallet.
- Trip monitoring.

Karakteristik bisnis:
- Low transaction frequency relative to MadWarung.
- Journey-based transactions.
- Situational user engagement.
- Considered purchasing decisions.
- Experience-oriented booking journey.

### MadTrans Roles

| Role | Fungsi |
|---|---|
| Owner Travel/Rental | Mengelola bisnis, armada, driver, jadwal, harga, pesanan, dan keuangan |
| Driver | Menjalankan perjalanan, penjemputan, pengantaran, dan konfirmasi operasional |
| Customer | Mencari layanan, memilih perjalanan, melakukan pemesanan, pembayaran, dan memantau perjalanan |

### Access Channel

| Role | Website | Mobile App |
|---|---|---|
| Owner Travel/Rental | Yes | No |
| Driver | No | Yes |
| Customer | No | Yes |

**Prinsip:** Manajemen bisnis operator dilakukan melalui website. Aplikasi mobile diperuntukkan bagi Driver dan Customer.

# 3. Public Experience

Public merupakan lapisan pengalaman publik yang menjadi pintu masuk ke ekosistem GoMad.

Public bukan vertical business tersendiri, melainkan kanal komunikasi, penemuan layanan, dan akuisisi pengguna.

Cakupan:
- Brand introduction.
- Product and service discovery.
- MadWarung introduction.
- MadTrans introduction.
- Campaign and promotion.
- Public information.
- Customer acquisition.
- Registration and onboarding entry points.

### Access Channel

| Role | Website | Mobile App |
|---|---|---|
| Public Visitor | Yes | No |
| Public User | Yes | No |

Public Website dapat mengarahkan pengguna ke aplikasi atau proses registrasi sesuai kebutuhan layanan.

Pengalaman Public harus konsisten dengan identitas GoMad, tetapi tidak harus memiliki struktur yang sama dengan dashboard operasional.

# 4. Product & Role Access Matrix

Matriks berikut menjadi acuan utama dalam menentukan platform dan interface yang dikembangkan.

| Product | Role | Website | Mobile App |
|---|---|---|---|
| GoMad Core | Admin | Yes | No |
| GoMad Core | Ops | Yes | No |
| MadWarung | Grosir | Yes | No |
| MadWarung | Retail Owner / Management | Yes | No |
| MadWarung | Penjaga Warung Retail | No | Yes |
| MadTrans | Owner Travel/Rental | Yes | No |
| MadTrans | Driver | No | Yes |
| MadTrans | Customer | No | Yes |
| Public | Visitor / Public User | Yes | No |

Ketentuan:
1. Tidak semua role membutuhkan aplikasi mobile.
2. Tidak semua role membutuhkan website.
3. Interface dikembangkan berdasarkan kebutuhan pekerjaan.
4. Website operasional dan aplikasi mobile tidak harus memiliki fitur yang identik.
5. Hak akses dan otorisasi ditentukan oleh backend, bukan oleh tampilan antarmuka.

# 5. Design Philosophy

GoMad Unified Design System mengadopsi prinsip:

**One Brand Identity, Multiple Design Languages.**

Artinya, seluruh produk menggunakan fondasi visual yang sama, tetapi pendekatan interface disesuaikan dengan kebutuhan pengguna.

Tiga pendekatan desain utama digunakan.

## 5.1 Operational Interface

**Design Language: Classic Minimalist**

Digunakan untuk:
- GoMad Core Admin.
- GoMad Core Ops.
- MadWarung Grosir.
- MadWarung Retail Management.
- MadTrans Owner Travel/Rental.

Karakteristik:
- Clean and structured.
- Information-oriented.
- Data-dense but readable.
- Functional navigation.
- Consistent spacing.
- Solid surfaces.
- Clear information hierarchy.
- Minimal decorative elements.

Prioritas:
1. Operational efficiency.
2. Data readability.
3. Fast information access.
4. Predictable interactions.
5. Productivity.

## 5.2 Frontline Operational Interface

**Design Language: Functional Minimalist**

Digunakan untuk:
- MadWarung Penjaga Warung Retail.
- MadTrans Driver.

Karakteristik:
- Mobile-first.
- Task-oriented.
- Simple navigation.
- Large touch targets.
- Clear action hierarchy.
- Minimal steps.
- High readability.
- Limited distractions.

Prioritas:
1. Task completion speed.
2. Ease of use.
3. Operational clarity.
4. Error prevention.
5. Reliable interaction.

Khusus aplikasi Driver, interface harus meminimalkan distraksi selama berkendara. Tindakan yang membutuhkan perhatian harus dilakukan ketika kendaraan berhenti di tempat aman.

## 5.3 Consumer Experience Interface

**Design Language: Experience-led Minimalist**

Digunakan untuk:
- MadTrans Customer.
- Public Website.

Referensi pengalaman: Airbnb.

Airbnb digunakan sebagai referensi pendekatan pengalaman pengguna, bukan sebagai template yang harus disalin.

Karakteristik:
- Experience-oriented.
- Visual-first presentation.
- Intuitive discovery.
- Generous whitespace.
- Clear typography.
- Editorial-style content hierarchy.
- Simple search and selection.
- Seamless booking flow.
- Contextual visual elements.

Prioritas:
1. User discovery.
2. Ease of exploration.
3. Decision-making support.
4. Booking simplicity.
5. Customer confidence.
6. Brand experience.

Glassmorphism dapat digunakan sebagai aksen visual pada elemen tertentu, tetapi bukan fondasi wajib seluruh interface.

# 6. Airbnb-Inspired Experience Direction

Pendekatan Airbnb menjadi referensi utama dalam membangun pengalaman MadTrans Customer dan Public.

Elemen pengalaman yang dapat diadaptasi:

## 6.1 Experience-led Discovery

Pengguna memulai dari kebutuhan perjalanan, bukan dari kumpulan formulir atau fitur.

Contoh:
- Mau bepergian ke mana?
- Dari mana perjalanan dimulai?
- Kapan ingin berangkat?
- Layanan seperti apa yang dibutuhkan?

## 6.2 Visual-first Presentation

Pilihan layanan disajikan dengan hierarki visual yang jelas.

Contoh:
- Travel cards.
- Destination cards.
- Operator profiles.
- Vehicle information.
- Service categories.
- Promotional content.

## 6.3 Intuitive Search Experience

Pencarian harus terasa natural dan sederhana.

Pengguna tidak perlu memahami struktur internal sistem untuk menemukan layanan.

## 6.4 Seamless Booking Journey

Alur perjalanan pengguna harus konsisten:

Discovery → Search → Explore → Select → Booking → Payment → Confirmation → Trip Experience.

## 6.5 Trust and Transparency

Informasi yang membantu pengguna mengambil keputusan harus mudah ditemukan.

Contoh:
- Harga.
- Jadwal.
- Kelas layanan.
- Informasi operator.
- Informasi pengemudi.
- Kebijakan pemesanan.
- Informasi pembayaran.
- Status perjalanan.

**Important:** GoMad mengadopsi prinsip UX Airbnb, bukan menyalin identitas merek, aset visual, atau komposisi antarmukanya secara literal.

# 7. Brand Visual Foundation

GoMad mempertahankan satu identitas merek lintas seluruh vertical business.

## 7.1 Brand Color Palette

| Token | Hex | Usage |
|---|---|---|
| Electric Lime | #A8E62E | Primary brand accent |
| Fresh Lime | #DFFF52 | Highlight |
| Vibrant Green | #258447 | Secondary brand |
| Deep Green | #174D38 | Strong brand surface |
| Sunny Yellow | #F6D83D | Accent |
| Cloud White | #F7FAF2 | Primary background |
| Pure White | #FFFFFF | Surface |
| Soft Gray | #F1F3EF | Secondary background |
| Border Gray | #E4E8E1 | Border |
| Muted Gray | #7D887D | Secondary text |
| Deep Olive | #263B2B | Primary text |

## 7.2 Semantic Colors

Semantic colors harus dibedakan dari warna identitas merek.

| Token | Hex | Meaning |
|---|---|---|
| Success | #258447 | Successful state |
| Warning | #D99A18 | Warning state |
| Error | #D64545 | Error state |
| Information | #3278B7 | Informational state |
| Neutral | #7D887D | Neutral state |

Status tidak boleh hanya dibedakan menggunakan warna. Gunakan label, ikon, dan konteks.

## 7.3 Color Distribution

Panduan komposisi warna:

Operational Interface:
- 75% neutral and white.
- 15% green.
- 5% yellow.
- 5% functional colors.

Consumer Experience:
- 50% neutral and white.
- 25% lime and green.
- 15% secondary green.
- 10% yellow.

Persentase ini merupakan pedoman visual, bukan aturan yang harus diterapkan secara kaku.

## 7.4 Color Accessibility

- Lime dan yellow tidak digunakan sebagai warna teks utama di atas background putih.
- Gunakan Deep Green atau Deep Olive untuk teks.
- Pastikan kontras memenuhi WCAG 2.2 AA.
- Normal text minimum contrast ratio 4.5:1.
- Large text minimum contrast ratio 3:1.

# 8. Typography System

Primary typeface: **Inter**

Alternative typeface: Plus Jakarta Sans.

Montserrat dapat digunakan untuk kebutuhan brand campaign dan marketing, tetapi bukan font utama interface.

| Style | Size | Weight |
|---|---:|---:|
| Display | 48px | 700 |
| H1 | 36px | 700 |
| H2 | 30px | 600 |
| H3 | 24px | 600 |
| H4 | 20px | 600 |
| Body Large | 18px | 400 |
| Body | 16px | 400 |
| Body Small | 14px | 400 |
| Caption | 12px | 400 |
| Label | 13px | 600 |

Typography harus memiliki hierarki konsisten, keterbacaan tinggi, dan penggunaan yang proporsional terhadap konteks interface.

# 9. Spacing, Grid & Layout

## 9.1 Spacing Scale

Gunakan basis kelipatan 4px.

4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80px.

## 9.2 Grid System

| Device | Columns | Padding |
|---|---:|---:|
| Mobile | 4 | 16px |
| Tablet | 8 | 24px |
| Desktop | 12 | 32px |
| Large Desktop | 12 | 40px |

Maximum content width: 1440px.

## 9.3 Border Radius

| Token | Value | Usage |
|---|---:|---|
| XS | 6px | Small controls |
| SM | 10px | Inputs and buttons |
| MD | 16px | Standard cards |
| LG | 20px | Large cards |
| XL | 24px | Consumer surfaces |
| Full | 999px | Pills and circular elements |

Operational interfaces cenderung menggunakan radius 6–16px.

Consumer interfaces dapat menggunakan radius 16–24px.

# 10. Glassmorphism Guidelines

Glassmorphism merupakan opsi visual, terutama untuk Consumer Experience Interface.

### Glass Tokens

| Token | Value |
|---|---|
| Glass Background | rgba(255,255,255,0.65) |
| Glass Strong | rgba(255,255,255,0.82) |
| Glass Border | rgba(255,255,255,0.75) |
| Blur | 16px |
| Strong Blur | 24px |
| Radius | 20px |
| Shadow | 0 8px 32px rgba(38,59,43,0.08) |

### Appropriate Usage

- Floating navigation.
- Search panels.
- Booking summaries.
- Destination selectors.
- Map controls.
- Promotional overlays.
- Selected cards.
- Floating CTA.
- Bottom sheets.

### Avoid Usage

- Dense data tables.
- Financial reports.
- Text-heavy forms.
- Long informational content.
- Complex operational interfaces.

Glassmorphism harus tetap memiliki fallback solid surface ketika efek transparansi atau blur tidak tersedia.

# 11. Shared Component Architecture

Komponen GoMad dibagi menjadi tiga kategori.

## 11.1 Core Components

Komponen lintas produk:
- Button.
- Input.
- Select.
- Checkbox.
- Radio.
- Badge.
- Tooltip.
- Modal.
- Drawer.
- Dropdown.
- Tabs.
- Pagination.
- Toast.
- Alert.
- Avatar.
- Breadcrumb.
- Navigation primitives.

## 11.2 Operational Components

Digunakan pada dashboard dan aplikasi operasional:
- Data Table.
- KPI Card.
- Filter Panel.
- Search Toolbar.
- Form Section.
- Status Indicator.
- Transaction List.
- Financial Summary.
- Inventory Table.
- Order Management.
- POS Cart.
- Trip Management.
- Fleet Management.
- Driver Status.
- Operational Timeline.

## 11.3 Consumer Experience Components

Digunakan pada MadTrans Customer dan Public:
- Destination Card.
- Travel Card.
- Service Category Card.
- Search Experience.
- Date Selector.
- Schedule Selector.
- Passenger Selector.
- Vehicle Class Selector.
- Booking Summary.
- Payment Card.
- Operator Profile.
- Driver Information.
- Trip Tracking Overlay.
- Floating Navigation.
- Promotional Banner.

Setiap komponen harus memiliki dokumentasi mengenai:
- Purpose.
- Anatomy.
- Variants.
- States.
- Interaction.
- Responsive behavior.
- Accessibility.
- Usage guidelines.

# 12. Website Design Guidelines

Website menjadi kanal utama untuk seluruh aktivitas manajemen dan operasional bisnis.

## 12.1 Operational Website

Digunakan oleh:
- GoMad Core Admin.
- GoMad Core Ops.
- MadWarung Grosir.
- MadWarung Retail Owner/Management.
- MadTrans Owner Travel/Rental.

Karakteristik:
- Desktop-first, responsive.
- Structured layouts.
- Sidebar or contextual navigation.
- Data tables.
- Filters and search.
- Form-oriented workflows.
- Dashboard analytics.
- Clear operational hierarchy.

## 12.2 Public Website

Public Website memiliki pendekatan berbeda dari dashboard.

Karakteristik:
- Experience-led.
- Responsive.
- Visual storytelling.
- Strong hero section.
- Service discovery.
- Clear CTA.
- Product and vertical introduction.
- Marketing and campaign content.

Website publik harus mampu memperkenalkan keseluruhan ekosistem GoMad sekaligus mengarahkan pengunjung ke layanan yang relevan.

# 13. Mobile App Design Guidelines

Mobile App hanya dikembangkan untuk tiga role.

| App | Role | Design Language |
|---|---|---|
| MadWarung App | Penjaga Warung Retail | Functional Minimalist |
| MadTrans App | Driver | Functional Minimalist |
| MadTrans App | Customer | Experience-led Minimalist |

## 13.1 MadWarung Retail Staff App

Fokus:
- POS.
- Product selection.
- Cart management.
- Checkout.
- Transaction confirmation.
- Daily transaction information.

Interface harus mengutamakan kecepatan transaksi, penggunaan satu tangan bila memungkinkan, dan target sentuh yang nyaman.

## 13.2 MadTrans Driver App

Fokus:
- Trip schedule.
- Booking information.
- Passenger information.
- Pickup and drop-off.
- Trip status.
- Navigation.
- Trip confirmation.
- COD confirmation, apabila berlaku.

Interface harus sederhana, jelas, dan mengurangi distraksi.

## 13.3 MadTrans Customer App

Fokus:
- Destination discovery.
- Search routes.
- Travel and rental selection.
- Schedule selection.
- Booking.
- Payment.
- E-ticket.
- Driver information.
- Trip tracking.
- Reviews.

Interface harus memberikan pengalaman perjalanan yang intuitif dan menyenangkan.

# 14. Responsive Design Standards

| Breakpoint | Range | Target |
|---|---|---|
| Mobile | 320–767px | Smartphone |
| Tablet | 768–1023px | Tablet |
| Desktop | 1024–1439px | Desktop and laptop |
| Large Desktop | 1440px+ | Large displays |

Ketentuan:
- Layout harus adaptif.
- Informasi penting tidak boleh hilang saat berpindah breakpoint.
- Navigation harus disesuaikan dengan ukuran layar.
- Table dapat menggunakan horizontal scrolling atau responsive transformation.
- Touch targets direkomendasikan minimum 44 × 44 CSS pixels.
- Consumer experience harus tetap konsisten di seluruh ukuran layar.

# 15. Accessibility & Usability

GoMad menargetkan WCAG 2.2 AA.

Standar:
- Readable typography.
- Sufficient color contrast.
- Keyboard accessibility untuk website.
- Visible focus states.
- Screen reader-friendly labels.
- Meaningful error messages.
- Clear validation.
- Reduced motion support.
- Accessible form controls.
- Status tidak hanya mengandalkan warna.

Usability harus diuji berdasarkan pekerjaan nyata masing-masing role, bukan hanya berdasarkan keseragaman visual.

# 16. Technical Design Architecture

Design System harus dikembangkan secara modular dan reusable.

Rekomendasi struktur:

```text
gomad-design-system/
│
├── foundations/
│   ├── colors/
│   ├── typography/
│   ├── spacing/
│   ├── radius/
│   ├── shadows/
│   ├── breakpoints/
│   └── accessibility/
│
├── themes/
│   ├── operational/
│   ├── frontline/
│   └── consumer/
│
├── components/
│   ├── core/
│   ├── operational/
│   ├── frontline/
│   └── consumer/
│
├── patterns/
│   ├── navigation/
│   ├── forms/
│   ├── tables/
│   ├── transactions/
│   ├── booking/
│   └── dashboards/
│
├── layouts/
│   ├── admin/
│   ├── operations/
│   ├── wholesale/
│   ├── retail-management/
│   ├── travel-owner/
│   ├── frontline/
│   └── public/
│
└── documentation/
    ├── foundations/
    ├── components/
    ├── patterns/
    └── implementation/
```

Struktur tersebut merupakan logical architecture. Implementasi fisik dapat menyesuaikan teknologi dan repository masing-masing produk.

### Implementation Principles

1. Shared foundations across all products.
2. Reusable components with documented variants.
3. Separate themes for different usage experiences.
4. Product-specific components remain modular.
5. Responsive behavior must be documented.
6. Accessibility must be built into components.
7. Design tokens must be centrally governed.
8. Avoid unnecessary component duplication.

# 17. Role-Based Interface Governance

Interface ditentukan berdasarkan product, role, dan access channel.

Contoh klasifikasi:

```text
GoMad Core
  ├── Admin → Website → Operational
  └── Ops   → Website → Operational

MadWarung
  ├── Grosir   → Website → Operational
  ├── Retail   → Website → Operational
  └── Penjaga  → App     → Frontline

MadTrans
  ├── Owner    → Website → Operational
  ├── Driver   → App     → Frontline
  └── Customer → App     → Consumer

Public
  └── Visitor  → Website → Consumer
```

### Governance Rules

- Active role determines the permitted interface.
- Product context determines available modules.
- Access channel determines the application surface.
- Backend authorization determines actual permissions.
- UI visibility is not a security mechanism.
- Multi-role users must have an explicit and secure role-switching mechanism where applicable.

# 18. Business Traffic & Experience Prioritization

High Traffic dan Low Traffic merupakan klasifikasi karakteristik transaksi bisnis.

Klasifikasi ini tidak otomatis menentukan frekuensi pengguna membuka aplikasi.

| Dimension | MadWarung | MadTrans |
|---|---|---|
| Business Traffic | High | Low |
| Transaction Pattern | Repetitive | Journey-based |
| Business Model | B2B | Travel/Rental Intermediary |
| User Activity | Procurement and retail transactions | Booking and trip activities |
| UX Priority | Efficiency | Experience |
| UI Priority | Operational clarity | Discovery and decision support |
| Main Platform | Website + Retail Staff App | Website + Customer/Driver App |

Konsekuensi desain:

**MadWarung**
- Minimize repetitive steps.
- Optimize frequent transactions.
- Support high-volume order management.
- Make operational data accessible.
- Prepare architecture for future B2B2C expansion.

**MadTrans**
- Simplify discovery.
- Make choices understandable.
- Reduce booking friction.
- Build customer trust.
- Provide clear trip information.

# 19. Design Governance & Quality Assurance

Semua produk GoMad harus mengikuti proses evaluasi desain.

Checklist:

- [ ] Brand identity consistency.
- [ ] Correct product and role classification.
- [ ] Correct website/mobile access allocation.
- [ ] Correct design language.
- [ ] Consistent design tokens.
- [ ] Reusable component implementation.
- [ ] Responsive behavior.
- [ ] Accessibility compliance.
- [ ] Interaction consistency.
- [ ] Error and empty states.
- [ ] Loading and success states.
- [ ] Documentation completeness.

Setiap perubahan Design System harus mempertimbangkan dampaknya terhadap seluruh produk.

# 20. Future Expansion & Scalability

Design System harus mampu mengakomodasi perkembangan GoMad tanpa merombak seluruh fondasi desain.

Prioritas pengembangan:

1. Current B2B MadWarung.
2. Current MadTrans Travel/Rental Intermediary.
3. Future MadWarung B2B2C.
4. Future product verticals.
5. Additional operational and consumer experiences.

Ekspansi bisnis tidak boleh membuat seluruh produk harus menggunakan bahasa desain yang sama secara kaku.

Shared foundations harus tetap stabil, sedangkan product-specific patterns dapat berkembang sesuai kebutuhan.

# 21. Documentation Roadmap

Dokumen ini merupakan fondasi utama untuk pengembangan dokumentasi berikutnya.

| Version | Document | Scope |
|---|---|---|
| v1.0 | Unified Design System Foundation | Business architecture and design philosophy |
| v1.1 | Design Tokens & Theme Specification | Colors, typography, spacing, themes |
| v1.2 | Shared Component Library Specification | Core, operational, frontline, consumer components |
| v1.3 | GoMad Core UI Guidelines | Admin and Ops |
| v1.4 | MadWarung UI Guidelines | Grosir, Retail, Retail Staff |
| v1.5 | MadTrans UI Guidelines | Owner, Driver, Customer |
| v1.6 | Public Website UI Guidelines | Public discovery and acquisition |
| v1.7 | Responsive & Mobile Interaction Guidelines | Responsive and mobile behavior |
| v1.8 | Developer & AI Agent Implementation Handbook | Technical implementation and governance |

# 22. Final Design Statement

GoMad bukan sekadar kumpulan aplikasi dengan tampilan yang seragam.

GoMad merupakan satu ekosistem bisnis dengan beberapa vertical yang memiliki karakteristik operasional, transaksi, dan pengalaman pengguna yang berbeda.

GoMad Core menjadi pusat kendali ekosistem.

MadWarung menghubungkan grosir dan retail melalui model B2B dengan orientasi transaksi berulang dan efisiensi operasional.

MadTrans menghubungkan operator travel/rental dengan penumpang melalui pengalaman perjalanan yang intuitif dan berorientasi pada pelanggan.

Public menjadi pintu masuk untuk memperkenalkan dan mengembangkan jangkauan ekosistem GoMad.

Karena itu, desain GoMad harus mampu menjaga satu identitas merek sekaligus memberikan pengalaman yang tepat bagi setiap pengguna.

**ONE BRAND. ONE ECOSYSTEM. MULTIPLE VERTICALS. MULTIPLE EXPERIENCES.**

---

**Document End**

GoMad Unified Design System v1.0 — Revised Business Architecture & Multi-Experience Design Foundation.