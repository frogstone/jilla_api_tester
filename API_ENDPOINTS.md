# API エンドポイント一覧

## 概要

iPadイベント受付アプリからLaravel APIへ送信するエンドポイント一覧です。

---

## データベース構造（参考）

### members テーブル（既存）

| カラム | 型 | 説明 |
|--------|-----|------|
| memID | int(11) | 主キー（AUTO_INCREMENT） |
| memName | varchar(100) | 会員名 |
| memNickname | varchar(100) | ニックネーム/屋号 |
| memMail | varchar(255) | メールアドレス |
| memZip | varchar(10) | 郵便番号 |
| memAddr | varchar(255) | 住所 |
| memTel | varchar(20) | 電話番号 |
| memFax | varchar(20) | FAX番号 |
| memMobile | varchar(20) | 携帯番号 |
| memUrl | varchar(255) | WebサイトURL |
| memSns | text | SNS情報 |
| memBirth | date | 生年月日 |
| memGender | tinyint(1) | 性別 |
| memDate | date | 入会日 |
| memNote | text | 備考 |
| delFlg | tinyint(1) | 削除フラグ（0:有効, 1:削除） |
| createdAt | datetime | 作成日時 |
| updatedAt | datetime | 更新日時 |

### events テーブル（新規）

| カラム | 型 | 説明 |
|--------|-----|------|
| id | bigint | 主キー（AUTO_INCREMENT） |
| name | varchar(255) | イベント名 |
| event_date | date | 開催日 |
| location | varchar(255) | 開催場所 |
| event_type | enum | セミナー/交流会/総会/その他 |
| description | text | 説明（NULL可） |
| max_capacity | int | 定員（NULL可） |
| owner_mem_id | int | オーナー（作成者）のmemID（FK → members.memID） |
| created_at | timestamp | 作成日時 |
| updated_at | timestamp | 更新日時 |

**重要**: イベントの閲覧・更新・削除は `owner_mem_id` が自分自身のイベントのみ可能です。

### event_attendances テーブル（新規）

| カラム | 型 | 説明 |
|--------|-----|------|
| id | bigint | 主キー（AUTO_INCREMENT） |
| event_id | bigint | イベントID（FK） |
| member_id | int | 会員ID（FK → members.memID） |
| checked_in_at | timestamp | チェックイン時刻 |
| checked_in_by | int | 受付担当のmemID（FK、NULL可） |
| device_id | varchar(100) | 受付端末識別子（NULL可） |
| created_at | timestamp | 作成日時 |
| updated_at | timestamp | 更新日時 |
| UNIQUE | | (event_id, member_id) |

---

## エンドポイント一覧

| Method | URL | 説明 |
|--------|-----|------|
| POST | `/api/auth/login` | ログイン |
| POST | `/api/auth/logout` | ログアウト |
| GET | `/api/auth/me` | ログイン中の会員情報取得 |
| POST | `/api/events` | イベント作成 |
| GET | `/api/events` | イベント一覧取得 |
| GET | `/api/events/{id}` | イベント詳細取得 |
| PUT | `/api/events/{id}` | イベント更新 |
| DELETE | `/api/events/{id}` | イベント削除 |
| POST | `/api/events/{id}/attendances` | 参加記録登録 |
| GET | `/api/events/{id}/attendances` | 参加者一覧取得 |
| DELETE | `/api/events/{id}/attendances/{attendanceId}` | 参加記録削除 |
| GET | `/api/members/{memID}/verify` | 会員検証 |

---

## 認証

### 1. ログイン

| 項目 | 内容 |
|------|------|
| **URL** | `POST /api/auth/login` |
| **認証** | 不要 |
| **Content-Type** | `application/json` |

#### リクエストボディ

```json
{
  "email": "member@example.com",
  "password": "password123"
}
```

#### レスポンス（成功）

```json
{
  "success": true,
  "data": {
    "token": "1|abc123xyz...",
    "member": {
      "memID": 100036,
      "memName": "恒田 英史",
      "memNickname": "bluehip studio",
      "memMail": "member@example.com"
    },
    "expiresAt": "2026-03-06T10:00:00.000Z"
  }
}
```

#### レスポンス（エラー）

```json
{
  "success": false,
  "message": "メールアドレスまたはパスワードが正しくありません"
}
```

---

### 2. ログアウト

| 項目 | 内容 |
|------|------|
| **URL** | `POST /api/auth/logout` |
| **認証** | Bearer Token |

#### レスポンス（成功）

```json
{
  "success": true,
  "message": "ログアウトしました"
}
```

---

### 3. ログイン中の会員情報取得

| 項目 | 内容 |
|------|------|
| **URL** | `GET /api/auth/me` |
| **認証** | Bearer Token |

#### レスポンス（成功）

```json
{
  "success": true,
  "data": {
    "memID": 100036,
    "memName": "恒田 英史",
    "memNickname": "bluehip studio",
    "memMail": "member@example.com"
  }
}
```

---

## イベント管理

### 4. イベント作成

| 項目 | 内容 |
|------|------|
| **URL** | `POST /api/events` |
| **認証** | Bearer Token |
| **Content-Type** | `application/json` |

#### リクエストボディ

```json
{
  "name": "第12回イラストレーターセミナー",
  "date": "2026-03-05",
  "location": "東京都渋谷区 協会会議室",
  "eventType": "seminar",
  "description": "プロのイラストレーターによる実践講座",
  "maxCapacity": 50
}
```

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| name | string | Yes | イベント名（最大255文字） |
| date | string | Yes | 開催日（YYYY-MM-DD形式） |
| location | string | Yes | 開催場所（最大255文字） |
| eventType | string | No | イベント種別（seminar/meetup/assembly/other） |
| description | string | No | 説明 |
| maxCapacity | int | No | 定員 |

#### レスポンス（成功）

```json
{
  "success": true,
  "message": "イベントを作成しました",
  "data": {
    "id": 123,
    "name": "第12回イラストレーターセミナー",
    "date": "2026-03-05",
    "location": "東京都渋谷区 協会会議室",
    "eventType": "seminar",
    "description": "プロのイラストレーターによる実践講座",
    "maxCapacity": 50,
    "owner": {
      "memID": 100036,
      "memName": "恒田 英史",
      "memNickname": "bluehip studio"
    },
    "createdAt": "2026-03-01T10:00:00.000Z"
  }
}
```

**注意**: イベント作成時、ログイン中の会員が自動的にオーナーとして設定されます。

---

### 5. イベント一覧取得

**自分がオーナーのイベントのみ取得されます。**

| 項目 | 内容 |
|------|------|
| **URL** | `GET /api/events` |
| **認証** | Bearer Token |

#### クエリパラメータ

| パラメータ | 型 | 説明 |
|-----------|-----|------|
| page | int | ページ番号（デフォルト: 1） |
| perPage | int | 1ページあたりの件数（デフォルト: 20） |
| from | string | 開始日（YYYY-MM-DD） |
| to | string | 終了日（YYYY-MM-DD） |
| eventType | string | イベント種別でフィルタ |

#### レスポンス（成功）

```json
{
  "success": true,
  "data": {
    "events": [
      {
        "id": 123,
        "name": "第12回イラストレーターセミナー",
        "date": "2026-03-05",
        "location": "東京都渋谷区 協会会議室",
        "eventType": "seminar",
        "attendanceCount": 25,
        "maxCapacity": 50,
        "owner": {
          "memID": 100036,
          "memName": "恒田 英史"
        }
      }
    ],
    "pagination": {
      "currentPage": 1,
      "lastPage": 5,
      "perPage": 20,
      "total": 98
    }
  }
}
```

---

### 6. イベント詳細取得

**自分がオーナーのイベントのみ取得可能です。**

| 項目 | 内容 |
|------|------|
| **URL** | `GET /api/events/{id}` |
| **認証** | Bearer Token |

#### レスポンス（成功）

```json
{
  "success": true,
  "data": {
    "id": 123,
    "name": "第12回イラストレーターセミナー",
    "date": "2026-03-05",
    "location": "東京都渋谷区 協会会議室",
    "eventType": "seminar",
    "description": "プロのイラストレーターによる実践講座",
    "maxCapacity": 50,
    "attendanceCount": 25,
    "owner": {
      "memID": 100036,
      "memName": "恒田 英史",
      "memNickname": "bluehip studio"
    },
    "createdAt": "2026-03-01T10:00:00.000Z",
    "updatedAt": "2026-03-01T10:00:00.000Z"
  }
}
```

#### レスポンス（権限エラー）

```json
{
  "success": false,
  "message": "このイベントを閲覧する権限がありません",
  "code": "FORBIDDEN"
}
```

---

### 7. イベント更新

**自分がオーナーのイベントのみ更新可能です。**

| 項目 | 内容 |
|------|------|
| **URL** | `PUT /api/events/{id}` |
| **認証** | Bearer Token |
| **Content-Type** | `application/json` |

#### リクエストボディ

```json
{
  "name": "第12回イラストレーターセミナー（改題）",
  "location": "オンライン開催に変更",
  "maxCapacity": 100
}
```

#### レスポンス（成功）

```json
{
  "success": true,
  "message": "イベントを更新しました",
  "data": {
    "id": 123,
    "name": "第12回イラストレーターセミナー（改題）",
    "date": "2026-03-05",
    "location": "オンライン開催に変更",
    "eventType": "seminar",
    "maxCapacity": 100,
    "updatedAt": "2026-03-02T15:00:00.000Z"
  }
}
```

#### レスポンス（権限エラー）

```json
{
  "success": false,
  "message": "このイベントを更新する権限がありません",
  "code": "FORBIDDEN"
}
```

---

### 8. イベント削除

**自分がオーナーのイベントのみ削除可能です。**

| 項目 | 内容 |
|------|------|
| **URL** | `DELETE /api/events/{id}` |
| **認証** | Bearer Token |

#### レスポンス（成功）

```json
{
  "success": true,
  "message": "イベントを削除しました"
}
```

#### レスポンス（権限エラー）

```json
{
  "success": false,
  "message": "このイベントを削除する権限がありません",
  "code": "FORBIDDEN"
}
```

#### レスポンス（エラー：参加者あり）

```json
{
  "success": false,
  "message": "参加者がいるイベントは削除できません",
  "code": "HAS_ATTENDANCES"
}
```

---

## 参加管理

### 9. 参加記録登録（メイン機能）

QRスキャン時に参加を登録します。

| 項目 | 内容 |
|------|------|
| **URL** | `POST /api/events/{id}/attendances` |
| **認証** | Bearer Token |
| **Content-Type** | `application/json` |

#### リクエストボディ

```json
{
  "memID": 100036,
  "checkedInAt": "2026-03-05T10:15:30.000Z",
  "deviceId": "iPad-Reception-01"
}
```

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| memID | int | Yes | 会員ID（members.memID） |
| checkedInAt | string | Yes | チェックイン時刻（ISO8601形式） |
| deviceId | string | No | 受付端末の識別子 |

#### レスポンス（成功）

```json
{
  "success": true,
  "message": "参加を記録しました",
  "data": {
    "id": 456,
    "eventId": 123,
    "member": {
      "memID": 100036,
      "memName": "恒田 英史",
      "memNickname": "bluehip studio"
    },
    "checkedInAt": "2026-03-05T10:15:30.000Z",
    "checkedInBy": {
      "memID": 100001,
      "memName": "運営担当者"
    }
  }
}
```

#### レスポンス（重複エラー）

```json
{
  "success": false,
  "message": "この会員は既に参加登録済みです",
  "code": "DUPLICATE_ATTENDANCE",
  "data": {
    "existingAttendance": {
      "id": 400,
      "checkedInAt": "2026-03-05T10:05:00.000Z"
    }
  }
}
```

#### レスポンス（会員不存在エラー）

```json
{
  "success": false,
  "message": "指定された会員が見つかりません",
  "code": "MEMBER_NOT_FOUND"
}
```

#### レスポンス（定員オーバーエラー）

```json
{
  "success": false,
  "message": "定員に達しているため参加登録できません",
  "code": "CAPACITY_EXCEEDED",
  "data": {
    "maxCapacity": 50,
    "currentCount": 50
  }
}
```

---

### 10. 参加者一覧取得

| 項目 | 内容 |
|------|------|
| **URL** | `GET /api/events/{id}/attendances` |
| **認証** | Bearer Token |

#### クエリパラメータ

| パラメータ | 型 | 説明 |
|-----------|-----|------|
| page | int | ページ番号 |
| perPage | int | 1ページあたりの件数 |
| format | string | `csv`を指定するとCSVダウンロード |

#### レスポンス（成功）

```json
{
  "success": true,
  "data": {
    "event": {
      "id": 123,
      "name": "第12回イラストレーターセミナー",
      "date": "2026-03-05"
    },
    "attendances": [
      {
        "id": 456,
        "member": {
          "memID": 100036,
          "memName": "恒田 英史",
          "memNickname": "bluehip studio"
        },
        "checkedInAt": "2026-03-05T10:15:30.000Z",
        "checkedInBy": {
          "memID": 100001,
          "memName": "運営担当者"
        }
      }
    ],
    "summary": {
      "totalCount": 25,
      "maxCapacity": 50
    }
  }
}
```

---

### 11. 参加記録削除

誤登録時の取り消し用です。

| 項目 | 内容 |
|------|------|
| **URL** | `DELETE /api/events/{id}/attendances/{attendanceId}` |
| **認証** | Bearer Token |

#### レスポンス（成功）

```json
{
  "success": true,
  "message": "参加記録を削除しました"
}
```

---

### 12. 会員検証

QRスキャン時に会員情報を事前検証します。

| 項目 | 内容 |
|------|------|
| **URL** | `GET /api/members/{memID}/verify` |
| **認証** | Bearer Token |

#### レスポンス（成功：有効な会員）

```json
{
  "success": true,
  "data": {
    "memID": 100036,
    "memName": "恒田 英史",
    "memNickname": "bluehip studio",
    "isActive": true,
    "memberSince": "2020-04-01"
  }
}
```

#### レスポンス（削除済み会員）

```json
{
  "success": false,
  "message": "この会員は退会済みです",
  "code": "MEMBER_DELETED"
}
```

---

## 追加提案エンドポイント

以下は今後の拡張として検討できるエンドポイントです。

### イベント統計

| Method | URL | 説明 |
|--------|-----|------|
| GET | `/api/events/{id}/stats` | イベント参加統計 |
| GET | `/api/stats/attendance` | 期間別参加統計 |

### 会員参加履歴

| Method | URL | 説明 |
|--------|-----|------|
| GET | `/api/members/{memID}/attendances` | 会員の参加履歴 |

### 事前登録（RSVPが必要な場合）

| Method | URL | 説明 |
|--------|-----|------|
| POST | `/api/events/{id}/reservations` | 事前参加登録 |
| GET | `/api/events/{id}/reservations` | 事前登録者一覧 |
| DELETE | `/api/events/{id}/reservations/{id}` | 事前登録キャンセル |

---

## 認証ヘッダー

すべての認証が必要なエンドポイントには以下のヘッダーを付与します。

```http
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

---

## HTTPステータスコード

| コード | 意味 |
|--------|------|
| 200 | 成功 |
| 201 | 作成成功 |
| 400 | リクエスト不正（バリデーションエラー） |
| 401 | 認証エラー |
| 403 | 権限エラー |
| 404 | リソース不存在 |
| 409 | 重複エラー（同一イベントに同一会員が登録済み） |
| 422 | バリデーションエラー |
| 500 | サーバーエラー |

---

## エラーレスポンス共通形式

```json
{
  "success": false,
  "message": "エラーメッセージ",
  "code": "ERROR_CODE",
  "errors": {
    "fieldName": ["バリデーションエラーの詳細"]
  }
}
```

### エラーコード一覧

| コード | 説明 |
|--------|------|
| VALIDATION_ERROR | バリデーションエラー |
| UNAUTHORIZED | 認証エラー |
| FORBIDDEN | 権限エラー |
| NOT_FOUND | リソース不存在 |
| DUPLICATE_ATTENDANCE | 参加重複 |
| MEMBER_NOT_FOUND | 会員不存在 |
| MEMBER_DELETED | 会員退会済み |
| CAPACITY_EXCEEDED | 定員超過 |
| HAS_ATTENDANCES | 参加者があるため削除不可 |

---

## 環境変数

```env
# iPadアプリ側
NEXT_PUBLIC_API_BASE_URL=https://your-laravel-api.com

# Laravel側
SANCTUM_STATEFUL_DOMAINS=localhost,your-ipad-app.vercel.app
SESSION_DOMAIN=.your-domain.com
```
