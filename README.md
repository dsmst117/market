# 쇼핑 리스트 앱

Supabase + Vercel로 구동하는 장보기 메모 애플리케이션입니다.

## 기능

- 장보기 아이템 추가 / 삭제
- 구매 완료 체크 (취소선 + 회색 처리)
- Supabase 데이터베이스와 **실시간 동기화 (Realtime)**
- 여러 기기 / 탭에서 동일한 데이터 자동 반영
- 연결 상태 표시 (연결 중 / 실시간 동기화 중 / 연결 실패)

## 기술 스택

- HTML / CSS / JavaScript (Vanilla)
- [Supabase](https://supabase.com) — PostgreSQL DB + Realtime
- [Vercel](https://vercel.com) — 서버리스 함수 + 호스팅

---

## Vercel 배포 설정

### 1. 환경 변수 추가

Vercel 대시보드 → 프로젝트 → **Settings > Environment Variables** 에서 다음 두 항목을 추가합니다:

```
SUPABASE_URL=your_supabase_project_url
SUPABASE_ANON_KEY=your_supabase_anon_key
```

### 2. Supabase 자격 증명 얻기

1. [Supabase](https://supabase.com) 로그인 → 프로젝트 선택
2. **Settings > API** 메뉴에서 확인:
   - **Project URL** → `SUPABASE_URL`
   - **anon public** 키 → `SUPABASE_ANON_KEY`

---

## 데이터베이스 설정 (Supabase SQL Editor)

### 테이블 생성

```sql
CREATE TABLE shopping_items (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  completed BOOLEAN DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL
);
```

### RLS (Row Level Security) 설정

```sql
-- RLS 활성화
ALTER TABLE shopping_items ENABLE ROW LEVEL SECURITY;

-- 모든 사용자 읽기 허용
CREATE POLICY "Enable read access for all users" ON shopping_items
  FOR SELECT USING (true);

-- 모든 사용자 삽입 허용
CREATE POLICY "Enable insert access for all users" ON shopping_items
  FOR INSERT WITH CHECK (true);

-- 모든 사용자 업데이트 허용
CREATE POLICY "Enable update access for all users" ON shopping_items
  FOR UPDATE USING (true);

-- 모든 사용자 삭제 허용
CREATE POLICY "Enable delete access for all users" ON shopping_items
  FOR DELETE USING (true);
```

### Realtime 활성화

Supabase 대시보드 → **Database > Replication** →  
`shopping_items` 테이블의 **Insert / Update / Delete** 이벤트를 활성화합니다.

---

## 로컬 개발

1. `.env.local` 파일 생성:

```
SUPABASE_URL=your_supabase_project_url
SUPABASE_ANON_KEY=your_supabase_anon_key
```

2. Vercel CLI로 실행:

```bash
vercel dev
```

> `vercel dev`는 `api/config.js` 서버리스 함수를 로컬에서도 에뮬레이션합니다.

---

## 아키텍처

```
사용자 브라우저
  → index.html 로드 (Vercel CDN)
  → GET /api/config  ← Vercel 서버리스 (환경변수에서 credentials 반환)
  → Supabase createClient() 초기화
  → shopping_items 테이블 CRUD (REST API)
  → Realtime 채널 구독 → 다른 탭/기기 변경사항 자동 반영
```
