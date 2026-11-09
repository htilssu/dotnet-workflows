# dotnet-workflows

Reusable GitHub Actions workflows cho các project .NET của `htilssu`.

> Repo private: chỉ các repo thuộc tài khoản `htilssu` gọi được (Settings → Actions → General → Access đã đặt là *Accessible from repositories owned by the user*).

## `dotnet-nuget.yml` — build, test, pack, push NuGet

```yaml
# .github/workflows/publish.yml trong repo của bạn
name: Build & publish package

on:
  push:
    branches: [main]
    tags: ['v*.*.*']
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  nuget:
    uses: htilssu/dotnet-workflows/.github/workflows/dotnet-nuget.yml@v1
    permissions:
      contents: read
      packages: write
    with:
      solution: MySolution.slnx
      pack-projects: |
        src/My.Library/My.Library.csproj
```

### Inputs

| Input | Mặc định | Ý nghĩa |
|---|---|---|
| `dotnet-version` | `10.0.x` | SDK cho `actions/setup-dotnet` (nhiều dòng được) |
| `solution` | rỗng | Solution/project để restore, build, test; rỗng = solution ở gốc repo |
| `pack-projects` | rỗng | Project cần pack, mỗi dòng một project; rỗng = pack cả solution (chỉ project có `IsPackable=true`) |
| `configuration` | `Release` | |
| `run-tests` | `true` | |
| `publish` | `true` | Có push package không; sự kiện `pull_request` không bao giờ push |
| `package-source` | `https://nuget.pkg.github.com/<owner>/index.json` | Nguồn push, ví dụ `https://api.nuget.org/v3/index.json` |
| `artifact-name` | `nuget-packages` | Tên artifact chứa `.nupkg` |

| Secret | Mặc định | Ý nghĩa |
|---|---|---|
| `NUGET_API_KEY` | `GITHUB_TOKEN` của repo gọi | API key cho `package-source` |

| Output | Ý nghĩa |
|---|---|
| `packages` | Tên các file `.nupkg` đã tạo, cách nhau bởi dấu cách |

### Hành vi

- Checkout toàn bộ lịch sử (`fetch-depth: 0`) để versioning theo git tag (MinVer, GitVersion…) hoạt động.
- Job `publish` yêu cầu quyền `packages: write`, nên job gọi phải cấp quyền này kể cả khi push sang nguồn khác.
- Push dùng `--skip-duplicate`: version đã tồn tại thì bỏ qua, không lỗi.
- Không có `.nupkg` nào được tạo → workflow lỗi kèm thông báo.

Push lên nuget.org:

```yaml
    with:
      package-source: https://api.nuget.org/v3/index.json
    secrets:
      NUGET_API_KEY: ${{ secrets.NUGET_ORG_API_KEY }}
```

## Version của workflow

Gọi bằng tag major `@v1`. Thay đổi tương thích ngược được phát hành bằng tag `v1.x.y` và dời tag `v1`; thay đổi phá vỡ (đổi tên/xoá input…) sang `v2`.

```bash
git tag v1.1.0 && git tag -f v1
git push origin v1.1.0 && git push -f origin v1
```

Cần cố định tuyệt đối thì gọi theo tag đầy đủ (`@v1.0.0`) hoặc commit SHA.
