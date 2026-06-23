# ZKAP 용어집

*이 문서의 [English](../en/GLOSSARY.md) 버전.*

> ZKAP 문서·저장소 전반에서 쓰이는 용어의 평이한 정의. 개요로 돌아가기:
> [README.md](./README.md) · [ARCHITECTURE.md](./ARCHITECTURE.md) ·
> [TRUST-MODEL.md](./TRUST-MODEL.md) · [REPOS.md](./REPOS.md).

네 그룹으로 나눕니다: 영지식 증명, 소셜 로그인, 계정 추상화, ZKAP 고유 용어.
표제어는 코드·문서에 등장하는 영어 그대로 두고, 설명을 한글로 답니다.

---

## 영지식 & 증명

- **Zero-knowledge proof (ZK proof, 영지식 증명).** 어떤 명제가 참임을, 그 외에는
  아무것도 드러내지 않고 증명하는 것. ZKAP에서는 로그인 토큰이 유효함을 *토큰을
  노출하지 않고* 보입니다.
- **ZK-SNARK.** *succinct*(작고) *non-interactive*(주고받음 없이 한 메시지)하며
  검증이 싼 영지식 증명.
- **Groth16.** ZKAP이 쓰는 특정 ZK-SNARK 시스템. 증명 크기가 고정·소형(256바이트 /
  필드 원소 8개)이고 온체인 검증이 쌉니다.
- **BN254 (alt_bn128).** 증명이 올라타는 타원곡선. EVM에 precompile(`0x06`/`0x07`/
  `0x08`)이 있어 온체인 검증이 감당 가능해집니다.
- **Poseidon.** 회로 *내부*에서 싸게 동작하도록 설계된 해시 함수(SHA-256보다 제약
  수가 훨씬 적음). Merkle 트리·anchor·nonce에 사용.
- **R1CS (Rank-1 Constraint System).** 명제가 SNARK용으로 컴파일되는 산술 형태.
  ZKAP 회로는 BN254 상의 R1CS.
- **arkworks.** 회로가 기반한 Rust 암호 라이브러리 생태계.
- **Trusted setup (신뢰 설정).** 회로마다 한 번 수행해 proving/verifying 키를 만드는
  단계. 비밀 난수("toxic waste")는 폐기해야 하며, 유출되면 거짓 증명이 가능해집니다 —
  그래서 다자간 ceremony로 수행합니다.
- **CRS / CRS bundle (Common Reference String).** SDK가 증명을 대조하는
  trusted-setup 출력: proving 키, verifying 키, Solidity verifier, 회로 config,
  manifest. *manifest* 참고.
- **Proving key (pk) / verifying key (vk) / prepared verifying key (pvk).** `pk`는
  증명 생성에, `vk`/`pvk`는 검증에 사용(`pvk`는 `vk`를 속도용으로 미리 계산한 형태).
- **Public inputs (공개 입력).** 증명자와 검증자가 모두 보는, 증명이 *그것에 대해*
  성립한다고 주장하는 값. ZKAP 증명은 proof당 8-요소 공개 입력 벡터를 가집니다.

## 소셜 로그인 (OIDC / JWT)

- **OIDC (OpenID Connect).** OAuth 2.0 위에 얹힌 신원 계층으로, "Google/Apple로
  로그인"이 이 위에 구축됩니다.
- **JWT (JSON Web Token).** 사용자가 로그인할 때 제공자가 발급하는 서명된 토큰. ZKAP은
  JWT가 진짜임을 노출 없이 증명합니다.
- **ID token.** 사용자가 누구인지 주장하는 OIDC JWT.
- **JWK (JSON Web Key).** 제공자의 *공개* 서명 키. 누구나 그 제공자가 서명한 JWT를
  검증할 수 있도록 공개됩니다.
- **kid (key ID).** JWT 헤더에서 어떤 JWK가 토큰에 서명했는지 가리키는 필드. 검증자가
  올바른 키를 고르게 합니다.
- **aud (audience).** 토큰이 발급된 대상 OAuth client ID. ZKAP은 이를 온체인에
  커밋하며(*hAudList* 참고), 증명은 커밋된 audience와 일치해야 합니다.
- **iss (issuer).** 토큰을 발급한 제공자(예: `https://accounts.google.com`).
- **sub (subject).** 제공자가 부여한 사용자의 안정적 식별자.
- **nonce.** 클라이언트가 로그인 요청에 넣고 제공자가 JWT 안에 그대로 돌려주는 값.
  ZKAP은 이를 특정 동작에 묶습니다(*zkNonce* 참고).
- **RS256 / RSA-2048.** 대부분의 OIDC 제공자가 JWT에 서명하는 알고리즘(RSA +
  SHA-256, 2048비트 키). 회로가 이 서명을 검증합니다.

## 계정 추상화 (ERC-4337)

- **Account abstraction (계정 추상화).** 계정을 단순 키쌍이 아니라 프로그래밍 가능한
  스마트 컨트랙트로 만들어, 누가 어떻게 서명할 수 있는지를 코드로 규정.
- **ERC-4337.** 프로토콜 변경 없이 계정 추상화를 가능하게 하는 이더리움 표준. 사용자
  의도가 공유 EntryPoint를 통해 흐릅니다.
- **EntryPoint.** UserOperation을 검증·실행하는 단일 정본 컨트랙트.
- **UserOperation (UserOp).** 계정 추상화의 "거래": 의도 + 서명 패킷으로, 번들러가
  EntryPoint에 제출.
- **Bundler (번들러).** UserOp을 모아 온체인 EntryPoint에 제출하는 오프체인 서비스.
- **Paymaster.** 사용자가 ETH 없이 거래하도록 가스를 후원할 수 있는 컨트랙트. ZKAP에선
  선택적.
- **Account factory.** 스마트 계정을 결정론적으로 배포하는 컨트랙트
  (`ZkapAccountFactory`).
- **Smart account (스마트 계정).** EOA가 아니라 컨트랙트인 지갑(여기서는
  `ZkapAccount`).
- **EOA (externally owned account).** 키쌍으로 통제되는 전통적 이더리움 계정.
- **CREATE2.** 컨트랙트 주소를 입력값으로부터 결정론적으로 만드는 opcode — 배포 전에
  주소를 알 수 있습니다.
- **Counterfactual address.** CREATE2 덕에 지갑이 배포되기 *전에도* 알 수 있고 자금을
  받을 수 있는 주소. 첫 UserOp이 이를 배포합니다.

## ZKAP 고유

- **ZKAP (Zero-Knowledge Authentication Protocol).** 이 허브가 문서화하는 프로토콜:
  소셜 로그인을 영지식으로 증명하고 그것을 이더리움 스마트 계정의 최상위 권한으로 사용.
- **ZkapAccount.** ZKAP 지갑의 중심에 있는 ERC-4337 스마트 계정 컨트랙트.
- **Master key (ZK-OAuth, 마스터 키).** 고가치 권한 — 소유권·복구·키 변경 — 으로,
  소셜 로그인 ZK 증명으로 사용.
- **TX key (TX 키).** 일상 권한 — 거래 — 으로, 기기 보관 패스키 또는 ECDSA 서명으로
  사용.
- **Passkey / WebAuthn / secp256r1.** 일상 거래 서명에 쓰는 기기 보관 자격증명(FIDO2 /
  WebAuthn, P-256 / secp256r1 곡선).
- **ECDSA.** EOA / address-key 서명에 쓰는 서명 방식.
- **Threshold anchor (임계 앵커).** 복구를 *k-of-n* OIDC 신원 집합에 묶어 온체인에
  등록하는 커밋먼트. Vandermonde / Shamir 방식 다항식 — 복구에 어떤 단일 issuer도
  필수가 아닙니다.
- **k-of-n.** 임계 규칙: 등록된 *n*개 신원 중 *k*개의 유효한 증명이 복구를 승인.
- **Vandermonde / Shamir scheme.** 임계 앵커 뒤에 있는 다항식 수학.
- **hAudList / audience hash.** 계정에 저장되는, 허용 audience(`aud`)의 Poseidon
  커밋먼트. 증명의 `h_aud_list`가 이와 일치해야 승인됩니다.
- **Issuer Merkle directory.** 신뢰 issuer의 RSA 공개키로 이루어진 온체인 Poseidon
  Merkle 트리. 회로가 서명 키가 그 멤버임을 증명(`PoseidonMerkleTreeDirectory`).
- **Merkle leaf / path / root.** 표준 Merkle 구성요소: *leaf*는 issuer-key 커밋먼트
  하나, *path*는 그 leaf가 *root* 아래 트리에 있음을 증명.
- **tree_height.** 회로/SDK Merkle path 길이(15). 온체인 컨트랙트 depth는
  `tree_height + 1`(16).
- **manifest.** CRS 번들 안의 `manifest.json`: 파일별 해시 + 선택적 서명. *단일 신뢰
  게이트* — 로더가 이를 검사하고, 이후 `prove()`는 번들을 신뢰하며 아무것도 재검증하지
  않습니다.
- **zkNonce.** `nonce = Poseidon(userOpHash, random)`. JWT nonce를 특정 UserOp에
  묶어, 증명이 그 동작 하나에만 유효하게 함 — 리플레이 방지.
- **Groth16Verifier / on-chain verifier.** 회로에서 *생성된* Solidity verifier로,
  BN254 pairing precompile을 통해 온체인에서 증명을 검사.
