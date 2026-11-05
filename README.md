# shadcn-ds-urbox

Repo tham chiếu cho thiết kế của team Product Design UrBox: Claude Design học token và component từ đây, và repo là nền cho Code Connect (map component set Figma với component code). **Không dùng cho production**: không có logic nghiệp vụ, API hay dữ liệu thật. Repo private vì giấy phép shadcn/studio Pro không cho chia sẻ công khai; code chỉ dùng shadcn/ui mã nguồn mở (MIT), không có component, block, registry hay MCP nào của shadcn/studio Pro.

## Phiên bản nguồn

| Nguồn | Phiên bản | File trong repo |
|---|---|---|
| Component Map | v1.0, bản giao Dev 03/11/2026 (quét lần 13, Figma library "Shadcn - Design System") | `docs/Component_Map_v1.0.source.xlsx` (gốc), `docs/Component_Map_v1.0_ten-trong-code.xlsx` (đã điền cột Tên trong code + sheet Bảng lệch) |
| Theme | `urbox-portal-theme.css` xuất từ Theme Generator của shadcn/studio, bản đính kèm 05/11/2026 | `src/styles/urbox-portal-theme.css` (giống hệt file gốc, không sửa) |
| Token trạng thái bổ sung | success, warning, info (Light + Dark), PD xác nhận 05/11/2026 | `src/styles/urbox-status-tokens.css` |
| shadcn/ui | CLI 4.23.0, style `new-york-v4`, base Radix UI | `components.json`, `src/components/ui/` |

## Font

Font của Portal là **Manrope**, theo SOP mục 0.4. Đây là ngoại lệ có chủ đích so với guideline Be Vietnam Pro của tổ chức. Manrope được nạp từ Google Fonts trong `index.html` (weight 400, 500, 600, 700; Google Fonts trả subset vietnamese theo unicode-range). Code không tự khai font-family: chữ lấy từ biến của file theme (`html` dùng `font-sans` → `--font-heading` → `--font-manrope` = "Manrope").

Lưu ý về file theme (không sửa, chỉ ghi lại): trong `@theme inline`, `--font-sans` và `--font-heading` trỏ vòng vào nhau; chuỗi chạy được nhờ `:root` trong `@layer base` đặt lại `--font-heading: var(--font-manrope)`. `body` khai `var(--font-body)` → `--font-geist` không được định nghĩa nên thừa kế Manrope từ `html`. Kết quả đo: `getComputedStyle(document.body).fontFamily` = `Manrope, sans-serif`.

## Token trạng thái và tông màu

PD xác nhận ngày 05/11/2026 bổ sung token `--success`, `--warning`, `--info` (kèm `-foreground`) cho Light và Dark. Token để ở file riêng `src/styles/urbox-status-tokens.css` (import sau file theme), không chèn vào `urbox-portal-theme.css`. Lý do: file theme là bản export của Theme Generator, lần export sau sẽ ghi đè mất token chèn tay; file riêng giữ được phép so diff với bản export. **Việc cần làm:** khi Theme Generator có các biến này thì thêm vào bản export, xoá file bổ sung và dòng `@import` trong `src/index.css`.

| Token | Light | Dark | Tương phản (WCAG 2.x) |
|---|---|---|---|
| success / -foreground | oklch(0.50 0.13 150) / oklch(0.985 0 0) | oklch(0.74 0.15 150) / oklch(0.205 0 0) | Solid 5.4:1 / 8.2:1 · Soft 4.9:1 / 7.1:1 |
| warning / -foreground | oklch(0.53 0.12 62) / oklch(0.985 0 0) | oklch(0.81 0.14 78) / oklch(0.205 0 0) | Solid 5.2:1 / 9.8:1 · Soft 4.8:1 / 8.2:1 |
| info / -foreground | oklch(0.53 0.16 250) / oklch(0.985 0 0) | oklch(0.74 0.12 245) / oklch(0.205 0 0) | Solid 5.0:1 / 7.9:1 · Soft 4.6:1 / 6.8:1 |

Đánh đổi: warning ở Light là hổ phách đậm (L 0.53) để chữ trắng đạt AA; vàng sáng sẽ không đạt. Soft destructive ở Light chỉ đạt 3.99:1 vì `--destructive` là token sẵn có của theme, repo không đổi.

**Tông màu** (`src/components/composed/tone.ts`): shadcn/ui không có variant Soft hay màu success/warning/info. Repo không sửa source shadcn mà phủ className bằng token lên variant gốc:

| Kiểu Figma | Code |
|---|---|
| Solid primary / secondary / destructive | `variant="default"` / `"secondary"` / `"destructive"` (giữ nguyên shadcn) |
| Solid success / warning / info | `variant="default"` + `className={tone("solid", màu)}` |
| Soft (mọi màu) | `variant="ghost"` + `className={tone("soft", màu)}`: nền màu 10%, chữ cùng màu |
| Outline / Ghost / Link (mọi màu) | `variant="outline"` / `"ghost"` / `"link"` + `className={tone(kiểu, màu)}` |
| Badge, Alert, Checkbox/Radio/Switch | `badgeTone()`, `alertTone()`, `controlTone()` theo cùng quy tắc |

Quyết định UX về Soft: dùng tông nhạt của chính màu đó thay vì `variant="secondary"`, vì `--secondary` là bề mặt tím nhạt với chữ gần đen, đọc như nút trung tính, và không mở rộng được sang Soft destructive/success/warning/info. Tooltip không nhận tông vì mũi tên cố định `bg-foreground` trong source shadcn (xem Bảng lệch).

## Quyết định đã chốt (05/11/2026)

| # | Nội dung | Lý do UX | Việc còn lại |
|---|---|---|---|
| 1 | Soft = nền màu 10% + chữ cùng màu (`tone("soft", màu)`) | Giữ nghĩa màu và một quy tắc cho mọi màu; `variant="secondary"` đọc như nút trung tính | Không |
| 2 | Bổ sung token success, warning, info | Trạng thái giao dịch, đối soát, hạn mức cần màu riêng, đạt AA | PD Lead thêm vào Theme Generator, rồi xoá `urbox-status-tokens.css` |
| 3 | Gộp Amber vào Warning (Alert) | Hai màu cảnh báo gần nhau không mang hai nghĩa khác nhau | PD xoá Amber trong Figma; code đã map Amber → warning |
| 4 | Tooltip chỉ giữ Style=Primary | Tooltip là chú thích phụ, không mang trạng thái; mũi tên shadcn cố định màu | PD xoá 5 Style còn lại trong Figma |
| 5 | Bỏ Gradient Button/Icon Button khỏi phạm vi Portal | Gradient cạnh tranh Solid primary, mờ thứ bậc hành động; không có token | PD đặt Giao Dev = Không cho CM-002, CM-008 ở Map v1.1; Dev chạy lại `npm run map:spec` |

Mục 3 đến 5 là thay đổi phía Figma/Map. Repo chưa xoá gì vì Map v1.0 vẫn ghi Giao Dev = Có; catalog tự cập nhật khi chạy lại script với Map mới.

## Quy ước thuộc tính

| Thuộc tính | Gắn ở đâu | Giá trị |
|---|---|---|
| `data-screen` | Phần tử gốc của một màn. Catalog: `<main data-screen="DS-Catalog">` | Tên màn |
| `data-ds` | Phần tử gốc của một instance component có trong Map | Đúng giá trị cột `data-ds` của Map |
| `data-ds-props` | Cùng phần tử với `data-ds` | `Property=Giá trị; Property=Giá trị` theo tên Figma, bỏ property/giá trị Figma-only; boolean ghi `=true`/`=false`; phần tử minh hoạ instance swap ghi thêm `Swap=lựa chọn` |
| `data-ds-new` | Phần tử ghép mới trên một màn, chưa có trong Map | Tên đề xuất. Catalog không dùng (0 phần tử) vì mọi phần tử đều có trong Map |

Tiêu đề khối và nhãn trên catalog là thẻ HTML thường, không gắn `data-ds`.

### Component không đưa `data-ds` xuống phần tử gốc

Thuộc tính được truyền như prop. Các trường hợp sau gắn trên thẻ bọc ngoài hoặc phần con hiển thị:

- **Root không render DOM** (Radix/Base UI): Dialog → `DialogContent`, AlertDialog → `AlertDialogContent`, Sheet → `SheetContent`, Drawer → `DrawerContent`, Popover → `PopoverContent`, Tooltip → `TooltipContent`, HoverCard → `HoverCardContent`, DropdownMenu → `DropdownMenuContent`, ContextMenu → `ContextMenuContent`. Combobox (CM-105) gắn trên `<div>` bọc ngoài.
- **Form** (CS-012): `Form` là FormProvider, gắn trên thẻ `<form>`.
- **InputOTP** (CM-075): props đi xuống thẻ input ẩn, nên gắn trên `<div>` bọc ngoài.
- **Sonner** (CM-102, CS-020): toast tạo bằng hàm `toast()`, gắn trên `<div>` bọc `Toaster`.
- **Component Figma gồm cả nhãn**: Input, Select, Textarea, Checkbox, Radio, Switch, Slider, Input Number, Date picker input, Combobox toolbar gắn trên `Field` bọc ngoài.
- **Biến thể ghép nhiều component**: Skeleton (CM-097), Pagination `With select`, Toggle Group `With input and select`, Progress `Top label=true`, Accordion `In card` (gắn trên `Card`), Chart `Interactive` (gắn trên `<div>` chứa nút chọn và biểu đồ).

### Overlay

Mọi overlay hiển thị ở trạng thái mở, mỗi cái trong một iframe riêng (`<iframe data-overlay-frame src="?frame=<Mã>&i=<chỉ số>&theme=light|dark">`), vì shadcn render overlay qua portal vào `body` với `position: fixed`. Phần tử `data-ds` của overlay nằm trong document của iframe; khi đếm phải tính cả các iframe (`scripts/check-catalog.mjs` làm việc này). Phần con chỉ sống trong menu mở (menu-item, Menubar item, Combobox Item) được đặt chung trong một overlay mở, nhãn dưới từng phần tử. Select có thêm một khung mở minh hoạ, không gắn `data-ds`.

## Cấu trúc

```
src/
  styles/urbox-portal-theme.css   nguồn token duy nhất (không sửa)
  index.css                       chỉ @import file theme
  components/ui/                  do npx shadcn add sinh ra (không sửa)
  components/composed/            component "Không có trong shadcn/ui", ghép từ component shadcn trong Map
  catalog/
    map.generated.json            sinh từ Map (scripts/map_to_spec.py)
    code-map.json                 cặp Figma → code (scripts/code_map.py)
    engine.ts                     sinh phần tử theo quy tắc catalog
    renderers/*.tsx               cách render từng Mã
    app.tsx                       trang catalog + trang con overlay
scripts/                          sinh spec, điền Map, kiểm catalog
docs/                             Component Map gốc, Map đã điền, Bảng lệch
```

Quy tắc sinh phần tử: tích các property variant (đã bỏ giá trị Figma-only); nếu tích vượt 24 thì render từng giá trị của mỗi property, các property khác giữ mặc định. Mỗi boolean render true và false (boolean con `- ...` bật boolean cha Show Top/Show Bottom). Mỗi instance swap render một lựa chọn khác mặc định (bật boolean icon tương ứng). Giá trị không có prop tương ứng vẫn render (bằng giao diện mặc định) và nhãn ghi "không có prop".

## Chạy local

Yêu cầu: Node 20+, Python 3 với `openpyxl` (chỉ cho script cập nhật Map), Chromium/Playwright (chỉ cho script kiểm).

```bash
npm install
npm run dev          # http://localhost:5173 , thêm ?theme=dark để xem Dark
npm run build
npx vite preview --port 4173
npm run check        # cần preview đang chạy: đếm data-ds, lỗi console, font, biến CSS, theme
```

## Cập nhật khi Map hoặc theme lên version mới

**Theme mới**: chép đè `src/styles/urbox-portal-theme.css` bằng file mới (không sửa tay); nếu bản export đã có success/warning/info thì xoá `urbox-status-tokens.css` và dòng import của nó, cập nhật bảng Phiên bản nguồn ở trên và `THEME_VERSION` trong `src/catalog/app.tsx`, chạy `npm run build` rồi `npm run check`. Nếu `--font-sans` trong file mới không dẫn tới Manrope: dừng lại, báo PD Lead.

**Map mới**:
1. Chép file vào `docs/`, sửa đường dẫn trong script `map:spec`, `map:fill`, `check` của `package.json`.
2. `npm run map:spec`: sinh lại `map.generated.json`; `code_map.py` báo lỗi "Thieu anh xa" cho mọi dòng, property, giá trị hoặc boolean mới chưa có cặp Figma → code.
3. Bổ sung cặp vào `scripts/code_map.py`; component mới: `npx shadcn add <tên>`, rồi thêm renderer theo Mã trong `src/catalog/renderers/`.
4. `npm run map:fill`: điền cột Tên trong code, sinh lại sheet Bảng lệch và `docs/Bang_lech.md`.
5. `npm run build`, chạy preview, `npm run check` phải ra đủ số và exit 0.

**Thêm component shadcn**: `npx shadcn add <tên>` (registry mặc định `ui.shadcn.com`, style `new-york-v4`). Nếu mạng chặn `ui.shadcn.com`, dựng mirror registry từ source GitHub của shadcn/ui và trỏ `REGISTRY_URL=<mirror>/r` (repo này được khởi tạo theo cách đó). Sau khi add, kiểm `git diff src/styles` phải rỗng.

## Component đã có

shadcn/ui (`src/components/ui`): accordion, alert, alert-dialog, avatar, badge, breadcrumb, button, button-group, calendar, card, carousel, chart, checkbox, collapsible, combobox, command, context-menu, dialog, drawer, dropdown-menu, field, form, hover-card, input, input-group, input-otp, item, kbd, label, menubar, navigation-menu, pagination, popover, progress, radio-group, scroll-area, select, separator, sheet, sidebar, skeleton, slider, sonner, spinner, switch, table, tabs, textarea, toggle, toggle-group, tooltip. `label` và hook `use-mobile` là dependency do CLI cài kèm (field, form, sidebar), không có khối riêng trên catalog.

Composed (`src/components/composed`): tone (helper tông màu), StatusIndicator, Steps/StepItem, Rating, InputNumber, Timeline, FormWizard, DatePicker, DashboardFooter.

Danh sách theo Map (107 dòng Giao Dev = Có):

| Mã | Component (Figma) | Loại | Tên trong code |
|---|---|---|---|
| CM-001 | button Group | Khớp shadcn | ButtonGroup |
| CM-002 | Gradient Icon Button | Nhiều set → một component | Button size="icon" |
| CM-003 | Solid Icon Button | Nhiều set → một component | Button size="icon" |
| CM-004 | Soft Icon Button | Nhiều set → một component | Button variant="ghost" size="icon" + className={tone("soft", màu)} |
| CM-005 | Outline Icon Button | Nhiều set → một component | Button variant="outline" size="icon" + className={tone("outline", màu)} |
| CM-006 | Ghost Icon Button | Nhiều set → một component | Button variant="ghost" size="icon" + className={tone("ghost", màu)} |
| CM-007 | Link Icon Button | Nhiều set → một component | Button variant="link" size="icon" + className={tone("link", màu)} |
| CM-008 | Gradient Button | Nhiều set → một component | Button |
| CM-009 | Solid Button | Nhiều set → một component | Button |
| CM-010 | Soft Button | Nhiều set → một component | Button variant="ghost" + className={tone("soft", màu)} |
| CM-011 | Outline Button | Nhiều set → một component | Button variant="outline" + className={tone("outline", màu)} |
| CM-012 | Ghost Button | Nhiều set → một component | Button variant="ghost" + className={tone("ghost", màu)} |
| CM-013 | Link Button | Nhiều set → một component | Button variant="link" + className={tone("link", màu)} |
| CM-023 | Accordion item | Phần con | AccordionItem |
| CM-024 | Accordion Trigger text | Phần con | AccordionTrigger |
| CM-025 | Accordion Content text | Phần con | AccordionContent |
| CM-026 | Accordion Trigger icon | Phần con | AccordionTrigger (ChevronDownIcon có sẵn) |
| CM-027 | Accordion | Khớp shadcn | Accordion |
| CM-028 | Avatar | Khớp shadcn | Avatar |
| CM-029 | Modifier / Indicator | Không có trong shadcn/ui | StatusIndicator (src/components/composed, ghép từ AvatarBadge; online=success, away=warning, busy=destructive) |
| CM-030 | Avatar group | Không có trong shadcn/ui | AvatarGroup (+ AvatarGroupCount) |
| CM-031 | Solid Badge | Nhiều set → một component | Badge |
| CM-032 | Soft Badge | Nhiều set → một component | Badge variant="outline" + className={badgeTone("soft", màu)} |
| CM-033 | Outline Badge | Nhiều set → một component | Badge variant="outline" + className={badgeTone("outline", màu)} |
| CM-034 | Dot Badge | Nhiều set → một component | Badge variant="outline" + chấm màu |
| CM-035 | Outline with icon | Nhiều set → một component | Badge variant="outline" + className={badgeTone("outline", màu)} + icon |
| CM-036 | Soft Dot Badge | Nhiều set → một component | Badge variant="outline" + className={badgeTone("soft", màu)} + chấm màu |
| CM-039 | Card | Khớp shadcn | Card |
| CM-040 | Dialog | Khớp shadcn | Dialog (data-ds trên DialogContent) |
| CM-041 | Alert Dialog | Khớp shadcn | AlertDialog (data-ds trên AlertDialogContent) |
| CM-042 | Collapsible | Khớp shadcn | Collapsible |
| CM-043 | Carousel | Khớp shadcn | Carousel |
| CM-044 | Pagination | Khớp shadcn | Pagination |
| CM-045 | Pagination Variants | Phần con | PaginationLink |
| CM-046 | Breadcrumb item | Phần con | BreadcrumbItem / BreadcrumbLink |
| CM-047 | Breadcrumb icon placeholder | Phần con | BreadcrumbSeparator |
| CM-048 | Breadcrumb | Khớp shadcn | Breadcrumb |
| CM-049 | Alert Outline | Nhiều set → một component | Alert + className={alertTone("outline", màu)} |
| CM-050 | Alert Solid | Nhiều set → một component | Alert + className={alertTone("solid", màu)} |
| CM-051 | Alert Soft | Nhiều set → một component | Alert + className={alertTone("soft", màu)} |
| CM-053 | Item | Khớp shadcn | Item |
| CM-054 | Tab item | Phần con | TabsTrigger |
| CM-057 | Steps / Step item | Không có trong shadcn/ui | StepItem (src/components/composed/steps.tsx) |
| CM-058 | Steps Horizontal | Không có trong shadcn/ui | Steps orientation="horizontal" (src/components/composed/steps.tsx) |
| CM-059 | Steps Vertical | Không có trong shadcn/ui | Steps orientation="vertical" (src/components/composed/steps.tsx) |
| CM-060 | Tooltip | Khớp shadcn | Tooltip (data-ds trên TooltipContent) |
| CM-062 | Progress | Khớp shadcn | Progress |
| CM-064 | Checkbox | Khớp shadcn | Checkbox (data-ds trên Field bọc ngoài) |
| CM-065 | Radio | Phần con | RadioGroupItem (data-ds trên Field bọc ngoài) |
| CM-067 | Switch | Khớp shadcn | Switch (data-ds trên Field bọc ngoài) |
| CM-069 | Rating | Không có trong shadcn/ui | Rating (src/components/composed/rating.tsx, ghép từ Toggle) |
| CM-071 | Slider | Khớp shadcn | Slider (data-ds trên Field bọc ngoài) |
| CM-072 | Input | Khớp shadcn | Input (data-ds trên Field bọc ngoài) |
| CM-073 | Input Group | Khớp shadcn | InputGroup |
| CM-074 | Select | Khớp shadcn | Select (data-ds trên Field bọc ngoài; SelectTrigger nhận size) |
| CM-075 | Input OTP | Khớp shadcn | InputOTP (data-ds trên thẻ bọc ngoài) |
| CM-076 | Input OTP Slot | Phần con | InputOTPSlot |
| CM-077 | File Input | Nhiều set → một component | Input type="file" |
| CM-078 | Textarea | Khớp shadcn | Textarea (data-ds trên Field bọc ngoài) |
| CM-079 | menu-item | Phần con | DropdownMenuItem |
| CM-080 | Dropdown Menu | Khớp shadcn | DropdownMenu (data-ds trên DropdownMenuContent) |
| CM-081 | Input Number | Không có trong shadcn/ui | InputNumber (src/components/composed/input-number.tsx, ghép từ InputGroup + InputGroupButton; data-ds trên Field bọc ngoài) |
| CM-082 | Popover | Khớp shadcn | Popover (data-ds trên PopoverContent) |
| CM-083 | Timeline | Không có trong shadcn/ui | Timeline (src/components/composed/timeline.tsx) |
| CM-084 | Form Wizard | Không có trong shadcn/ui | FormWizard (src/components/composed/form-wizard.tsx, ghép từ Steps + Card + Button) |
| CM-085 | Calendar | Khớp shadcn | Calendar |
| CM-086 | Date picker input | Không có trong shadcn/ui | DatePicker (src/components/composed/date-picker.tsx, ghép từ Popover + Button + Calendar; data-ds trên Field bọc ngoài) |
| CM-089 | Dashboard Footer | Không có trong shadcn/ui | DashboardFooter (src/components/composed/dashboard-footer.tsx) |
| CM-091 | Menubar item | Phần con | MenubarItem |
| CM-093 | NavigationMenu list | Phần con | NavigationMenuLink |
| CM-094 | NavigationMenu Popover | Phần con | NavigationMenuContent |
| CM-095 | Separator | Khớp shadcn | Separator |
| CM-097 | Skeleton | Khớp shadcn | Skeleton |
| CM-098 | Sheet | Khớp shadcn | Sheet (data-ds trên SheetContent) |
| CM-099 | Drawer | Khớp shadcn | Drawer (data-ds trên DrawerContent) |
| CM-100 | Scroll Area | Khớp shadcn | ScrollArea |
| CM-101 | Spinner | Khớp shadcn | Spinner |
| CM-102 | Sonner | Khớp shadcn | Toaster + toast() (shadcn đặt tên file sonner.tsx, component là Toaster) |
| CM-103 | Combobox toolbar | Phần con | ComboboxInput / ComboboxChips (data-ds trên Field bọc ngoài) |
| CM-104 | Combobox Item | Phần con | ComboboxItem |
| CM-105 | Combobox | Khớp shadcn | Combobox (data-ds trên thẻ bọc ngoài, Combobox root không render DOM) |
| CM-106 | Command | Khớp shadcn | Command |
| CM-107 | Kbd | Khớp shadcn | Kbd |
| CM-108 | Hover card | Khớp shadcn | HoverCard (data-ds trên HoverCardContent) |
| CM-109 | table-header | Phần con | TableHead |
| CM-110 | table-cell | Phần con | TableCell |
| CM-111 | Sidebar menu item | Phần con | SidebarMenuItem (+ SidebarMenuButton) |
| CM-112 | Sidebar | Khớp shadcn | Sidebar |
| CM-113 | Toggle | Khớp shadcn | Toggle |
| CM-114 | Toggle Group | Khớp shadcn | ToggleGroup |
| CM-116 | Radar Charts New | Nhiều set → một component | ChartContainer + Recharts RadarChart |
| CM-117 | Area Chart New | Nhiều set → một component | ChartContainer + Recharts AreaChart |
| CM-118 | Radial chart New | Nhiều set → một component | ChartContainer + Recharts RadialBarChart |
| CM-119 | Line chart New | Nhiều set → một component | ChartContainer + Recharts LineChart |
| CM-120 | Pie Chart New | Nhiều set → một component | ChartContainer + Recharts PieChart |
| CM-121 | Bar Chart New | Nhiều set → một component | ChartContainer + Recharts BarChart |
| CS-012 | Form | Khớp shadcn | Form (react-hook-form; data-ds trên thẻ <form>, Form là FormProvider không render DOM) |
| CS-013 | menu-label | Phần con | DropdownMenuLabel |
| CS-015 | Context Menu | Khớp shadcn | ContextMenu (data-ds trên ContextMenuContent) |
| CS-017 | Menubar | Khớp shadcn | Menubar |
| CS-018 | Menubar menu | Phần con | MenubarContent |
| CS-019 | Navigation Menu | Khớp shadcn | NavigationMenu |
| CS-020 | Sonner item | Phần con | toast() (một toast trong Toaster; data-ds trên thẻ bọc ngoài) |
| CS-021 | Combobox List | Phần con | ComboboxList |
| CS-023 | Table | Khớp shadcn | Table |
| CS-024 | Sidebar Menu Item Group | Phần con | SidebarGroup |
| CS-030 | Field | Khớp shadcn | Field |
