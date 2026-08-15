# 래미안베라힐즈 민원관리시스템

개발가이드.txt 를 기준으로 만든 웹앱입니다. 현재 Firebase(Firestore + Authentication)와
연동되어, 여러 사람·여러 기기에서 실시간으로 같은 데이터를 보고 편집할 수 있습니다.

## 포함 파일
- `index.html` — 메인 앱 (대시보드 / 민원이력 / 동·호수관리 / 공용부관리 / 민원유형관리 / 백업복원)
- `firebase-bundle.js` — Firebase(Firestore, Authentication) 연동 코드 번들
- `xlsx.full.min.js` — 엑셀 일괄등록 기능에 쓰이는 라이브러리
- `manifest.json` — 홈 화면에 앱처럼 추가하기 위한 PWA 설정
- `icon-192.png`, `icon-512.png` — 앱 아이콘(파비콘)

이 5개 파일은 항상 같은 폴더(같은 GitHub 저장소)에 함께 있어야 합니다.

## 실시간 동기화 (Firestore)
데이터(세대, 공용부, 민원유형, 민원이력)는 이제 Firebase Firestore에 저장됩니다.
누군가 민원을 등록하거나 수정하면, 같은 사이트에 접속한 다른 사람 화면에도 자동으로
바로 반영됩니다. 최초 배포 후 관리자가 처음 로그인하면, 데이터가 비어있는 경우에 한해
1회만 기본 데이터(1,305세대, 공용부, 민원유형 목록)를 자동으로 채웁니다.

## 관리자 계정 (Firebase Authentication)
- 관리자 로그인은 Firebase Authentication(이메일/비밀번호) 계정으로 이루어집니다.
- 로그인하지 않으면 조회만 가능하고, 로그인하면 등록/수정/삭제가 가능합니다.
- 비밀번호를 잊었거나 바꾸고 싶으면, "백업/복원" 화면 하단의 "비밀번호 재설정 메일
  보내기" 버튼을 누르면 로그인된 관리자 이메일로 재설정 링크가 발송됩니다.
- 관리자 계정 추가/삭제는 Firebase 콘솔의 Authentication > Users 메뉴에서 관리합니다.

## Firestore 보안 규칙
Firebase 콘솔 > Firestore Database > 규칙(Rules) 탭에 아래 내용이 적용되어 있어야
합니다(비로그인 사용자는 조회만, 로그인한 관리자만 쓰기 가능):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

## 세대 엑셀 재일괄등록
"동·호수관리" 화면의 "엑셀 일괄등록" 버튼으로 언제든 엑셀 파일을 다시 올릴 수
있습니다. 파일 첫 시트에 `연번`(선택), `동`, `호`(또는 `호수`) 열이 있으면
자동으로 인식합니다. "기존 유지+추가"와 "전체 교체" 두 가지 방식을 선택할 수
있습니다.

## 백업 / 복원
"백업/복원" 화면에서 전체 데이터를 JSON 파일로 내려받거나(백업), JSON 파일을
올려 현재 데이터를 덮어쓸 수 있습니다(복원). 복원 시 기존 문서의 고유 ID를 그대로
유지하므로, 민원이력이 참조하는 공용부 연결 정보도 안전하게 보존됩니다.

## 배포 방식 (참고)
1. **GitHub**: 이 폴더의 5개 파일을 GitHub 저장소에 올립니다.
2. **Netlify**: 해당 저장소를 연결하면 자동으로 배포되고 `xxx.netlify.app` 주소가
   생성됩니다. 파일을 변경할 때마다 GitHub에 다시 업로드하면 Netlify가 자동으로
   재배포합니다.
3. **Firebase**: Firestore(실시간 데이터베이스) + Authentication(관리자 로그인)을
   사용합니다. 설정값은 `index.html` 상단의 `firebaseConfig`에 이미 반영되어
   있습니다.
