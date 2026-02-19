# GitHub 레포지토리 생성 및 PR 작업 과정 정리

이 문서는 `test-repo` 레포지토리를 생성하고, README.md를 추가한 뒤 PR을 올리기까지의 전체 명령어와 그 의미를 정리한 것입니다.

---

## 1단계: GitHub 레포지토리 생성

```bash
gh repo create test-repo --public --confirm
```

| 옵션 | 의미 |
|------|------|
| `gh repo create` | GitHub CLI를 사용하여 원격 레포지토리를 생성 |
| `test-repo` | 레포지토리 이름 |
| `--public` | 공개 레포지토리로 생성 |
| `--confirm` | 확인 프롬프트 없이 바로 생성 (deprecated되었지만 동작함) |

---

## 2단계: 클론 및 feat 브랜치 생성

```bash
git clone https://github.com/Baek-SeungHun/test-repo.git
cd test-repo
git checkout -b feat
```

| 명령어 | 의미 |
|--------|------|
| `git clone <url>` | 원격 레포지토리를 로컬로 복제 |
| `cd test-repo` | 클론된 디렉토리로 이동 |
| `git checkout -b feat` | `feat`이라는 새 브랜치를 생성하고 전환 |

> 빈 레포지토리를 클론했기 때문에 `warning: You appear to have cloned an empty repository.` 경고가 출력되었습니다.

---

## 3단계: README.md 작성 및 커밋

```bash
git add README.md
git commit -m "feat: add README.md"
```

| 명령어 | 의미 |
|--------|------|
| `git add README.md` | README.md 파일을 스테이징 영역에 추가 |
| `git commit -m "..."` | 스테이징된 변경사항을 커밋으로 기록 |

---

## 4단계: feat 브랜치 푸시

```bash
git push -u origin feat
```

| 옵션 | 의미 |
|------|------|
| `git push` | 로컬 커밋을 원격 저장소에 업로드 |
| `-u` (`--set-upstream`) | 로컬 브랜치와 원격 브랜치의 추적 관계를 설정 (이후 `git push`만으로 푸시 가능) |
| `origin feat` | `origin` 원격의 `feat` 브랜치로 푸시 |

---

## 5단계: main 브랜치 생성 (문제 발생)

```bash
git checkout --orphan main
git commit --allow-empty -m "initial commit"
git push -u origin main
```

| 명령어 | 의미 |
|--------|------|
| `git checkout --orphan main` | 기존 히스토리와 연결되지 않은 독립적인 `main` 브랜치 생성 |
| `git commit --allow-empty` | 파일 변경 없이 빈 커밋을 허용하여 생성 |
| `git push -u origin main` | main 브랜치를 원격에 푸시 |

> **문제**: `--orphan`으로 만든 브랜치는 `feat`과 공통 히스토리(common ancestor)가 없습니다. Git에서 PR은 두 브랜치의 공통 조상을 기준으로 diff를 계산하기 때문에, 공통 히스토리가 없으면 PR 생성이 불가능합니다.

---

## 6단계: 기본 브랜치 설정

```bash
gh repo edit --default-branch main
```

| 명령어 | 의미 |
|--------|------|
| `gh repo edit` | GitHub 레포지토리 설정을 수정 |
| `--default-branch main` | 기본 브랜치를 `main`으로 변경 |

> PR의 base 브랜치가 될 `main`을 기본 브랜치로 설정해야 합니다.

---

## 7단계: PR 생성 시도 (실패)

```bash
gh pr create --base main --head feat --title "feat: add README.md" --body "..."
```

**에러 메시지:**
```
pull request create failed: GraphQL: The feat branch has no history in common with main
```

> **원인**: 5단계에서 `--orphan`으로 main을 만들었기 때문에, `feat`과 `main`은 완전히 별개의 히스토리를 가진 브랜치가 되어 PR을 만들 수 없었습니다.

---

## 8단계: main 브랜치 재구성 (문제 해결)

```bash
git checkout main
git rm README.md
git commit --amend --allow-empty -m "initial commit"
git push origin main --force
```

| 명령어 | 의미 |
|--------|------|
| `git rm README.md` | README.md를 Git 추적에서 제거 |
| `git commit --amend --allow-empty` | 직전 커밋을 수정하여 빈 커밋으로 만듦 |
| `git push --force` | 원격의 히스토리를 로컬 히스토리로 강제 덮어쓰기 |

> main을 빈 초기 커밋 상태로 만들어, feat에서 README.md를 추가하는 변경사항이 명확히 보이도록 했습니다.

---

## 9단계: feat 브랜치를 main 기반으로 재생성

```bash
git checkout -B feat main
```

| 옵션 | 의미 |
|------|------|
| `-B` | 이미 존재하는 브랜치를 강제로 재설정 (`-b`는 이미 존재하면 에러) |
| `feat main` | `main` 브랜치를 시작점으로 `feat` 브랜치를 생성 |

> 이렇게 하면 `feat`이 `main`의 히스토리를 공유하게 되어 PR 생성이 가능해집니다.

---

## 10단계: README.md 재커밋 및 푸시

```bash
git add README.md
git commit -m "feat: add README.md"
git push origin feat --force
```

| 명령어 | 의미 |
|--------|------|
| `git add` + `git commit` | README.md를 다시 스테이징하고 커밋 |
| `git push --force` | 이전에 푸시된 feat 브랜치의 히스토리를 새 히스토리로 강제 교체 |

---

## 11단계: PR 생성 (성공)

```bash
gh pr create --base main --head feat \
  --title "feat: add README.md" \
  --body "..."
```

| 옵션 | 의미 |
|------|------|
| `--base main` | PR의 대상(병합될) 브랜치 |
| `--head feat` | PR의 소스(변경사항이 있는) 브랜치 |
| `--title` | PR 제목 |
| `--body` | PR 본문 (Markdown 지원) |

> `feat`이 `main`에서 분기되었으므로 공통 히스토리가 존재하여 PR이 정상 생성되었습니다.

---

## 핵심 교훈

| 문제 | 원인 | 해결 |
|------|------|------|
| PR 생성 실패 | `--orphan`으로 만든 브랜치는 공통 히스토리가 없음 | `feat`을 `main`에서 분기하여 재생성 |
| 빈 레포에 PR 올리기 | 빈 레포지토리에는 기본 브랜치가 없음 | main 브랜치를 먼저 만들고 기본 브랜치로 설정 |

**결론**: PR을 만들려면 두 브랜치가 반드시 **공통 조상 커밋**을 공유해야 합니다. 새 레포에서 작업할 때는 `main`을 먼저 만들고, 그 위에서 feature 브랜치를 분기하는 것이 올바른 순서입니다.
