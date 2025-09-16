# AI CMOにおけるプロンプトとナレッジの連携フロー

このドキュメントは、AI CMOがどのようにプロンプト、ナレッジ、サマリー、教育コンテンツ、クリエイティブパーツを連携させて、最終的な広告アウトプットを生成するかを可視化します。

## 1. 全体像

AI CMOは、以下の主要なステップを経て広告コンテンツを生成します。

1.  **商品サマリー生成**: 商品情報から詳細なマーケティングリサーチレポートを作成。
2.  **教育コンテンツ生成**: 商品サマリーを基に、顧客の潜在ニーズを顕在化させる教育コンテンツの材料を作成。
3.  **クリエイティブパーツ生成**: プロジェクト情報、AIペルソナ（AIの役割設定）、クリエイティブ要素のマスターナレッジを基に、広告表現のパーツを量産。
4.  **広告アウトプット生成**: 上記で生成された情報と、話者ペルソナ、構成パターンを組み合わせて、最終的な広告台本（ショート動画、LPなど）を作成。
5.  **追加処理**: 生成された広告アウトプットに対して、アノテーションや法的チェックを実施。

これらのステップは、特定のプロンプトテンプレートと、AIが参照するナレッジベースを組み合わせることで実現されます。

## 2. 主要なプロンプトとナレッジの連携

### 2.1. プロンプトテンプレートの読み込み

すべてのプロンプトテンプレートおよびナレッジファイルは、`src/lib/utils/promptLoader.ts` の `loadMarkdownFile` 関数を通じてMarkdownファイルから動的に読み込まれます。

### 2.2. 商品サマリーの生成

*   **目的**: 特定の商品に関する詳細なマーケティングリサーチレポートを作成。
*   **関連ファイル**:
    *   `src/lib/prompts/summaryPrompts.ts`: `generateProductSummaryPrompt` 関数。
    *   `src/lib/prompt_v2/ppt01_product_summery.md`: 商品サマリー生成のベースとなるプロンプトテンプレート。
*   **連携フロー**:
    1.  `generateProductSummaryPrompt` が `ppt01_product_summery.md` を読み込む。
    2.  ユーザーが入力した `productInfo` (商品名、URL、テキストデータなど) をテンプレート内のプレースホルダーに挿入。
        *   **例**: `{{productInfo}}` に「商品名: AIアシスタント、URL: example.com/ai-assistant」が挿入される。
    3.  必要に応じて `modificationInstruction` (修正指示) を追加。
    4.  AIがこのプロンプトを基に商品サマリーを生成。
*   **ナレッジの役割**: `ppt01_product_summery.md` 自体が、プロのマーケティングリサーチャーとしての役割設定、詳細な調査計画、情報抽出、矛盾点分析、そして「特徴」「メリット」「実績」「権威性」「オファー」といった具体的な出力フォーマットを定義するナレッジとして機能します。

### 2.3. 教育コンテンツの生成

*   **目的**: 商品サマリーを基に、顧客の潜在的なニーズを顕在化させるための教育コンテンツの材料をリサーチ・作成。
*   **関連ファイル**:
    *   `src/lib/prompts/educationPrompts.ts`: `generateEducationContentPrompt` 関数。
    *   `src/lib/prompt_v2/ppt02_education_summery.md`: 教育コンテンツ生成のベースとなるプロンプトテンプレート。
*   **連携フロー**:
    1.  `generateEducationContentPrompt` が `ppt02_education_summery.md` を読み込む。
    2.  生成済みの `productSummaryContent` をテンプレート内のプレースホルダーに挿入。
        *   **例**: `{{productSummaryContent}}` に前のステップで生成された商品サマリーが挿入される。
    3.  必要に応じて `modificationInstruction` (修正指示) を追加。
    4.  AIがこのプロンプトを基に教育コンテンツの材料を生成。
*   **ナレッジの役割**: `ppt02_education_summery.md` 自体が、プロのコンテンツマーケターとしての役割設定、目的（顧客の課題・欲求の顕在化）、詳細な実行ステップ（調査計画、検索実行、情報抽出）、そして「重要性・必要性」「自分ごと化を促す客観的データ」「意外な事実・トリビア」といった出力フォーマットを定義するナレッジとして機能します。

### 2.4. クリエイティブパーツの生成

*   **目的**: 広告コミュニケーションに使える表現のパーツを量産。
*   **関連ファイル**:
    *   `src/lib/prompts/creativePartsPrompts.ts`: `generateCreativePartsPrompt` 関数と `getCreativePartsMaster` 関数。
    *   `src/lib/prompt_v2/ppt03_creative_parts.md`: クリエイティブパーツ生成のベースとなるプロンプトテンプレート。
    *   `src/lib/knowledge_v2/creative_parts_elements/creative_parts_master.md`: クリエイティブパーツのマスターナレッジ（「広告クリエイティブ大全：統合版」）。
*   **連携フロー**:
    1.  `getCreativePartsMaster` が `creative_parts_master.md` を読み込む。
    2.  `generateCreativePartsPrompt` が `ppt03_creative_parts.md` を読み込む。
    3.  `projectInformation` (商品、顧客、戦略)、`creativePartsMaster` (読み込まれた `creative_parts_master.md` の内容)、`aiPersona` (AIの役割設定) をテンプレート内のプレースホルダーに挿入。
        *   **例**: `{{creativePartsMaster}}` に「広告クリエイティブ大全：統合版」の内容が挿入される。
    4.  必要に応じて `modificationInstruction` (修正指示) を追加。
    5.  AIがこのプロンプトとナレッジを基にクリエイティブパーツを生成。
*   **ナレッジの役割**: `creative_parts_master.md` は、フック、問題提起、解決策、信頼構築、クロージング、高度な構成といった広告のストーリーフローに沿って、具体的なクリエイティブ要素と表現例を網羅的に提供する、AIが参照すべき最も重要なナレッジベースです。

### 2.5. 広告アウトプットの生成

*   **目的**: ショート動画、YouTube広告、LPなどの最終的な広告台本を生成。
*   **関連ファイル**:
    *   `src/lib/prompts/outputPrompts.ts`: `generateAdOutputPrompt` 関数と `loadSpeakerPersona` 関数。
    *   `src/lib/prompt_v2/ppt04-01_short_ads.md` (例): 各広告フォーマットに対応するプロンプトテンプレート。
    *   `src/lib/knowledge_v2/speaker_persona/*.md` (例: `persona05_developer.md`): AIが憑依する話者のペルソナ定義ナレッジ。
    *   `src/lib/knowledge_v2/structure/*.md` (例: `structure01_short_ads.md`): 各広告フォーマットのヒット構成パターンナレッジ。
*   **連携フロー**:
    1.  `generateAdOutputPrompt` が `outputType` に応じて適切な `ppt04-xx_*.md` テンプレートを読み込む。
    2.  `loadSpeakerPersona` が指定された話者ペルソナファイル (例: `persona05_developer.md`) を読み込む。
    3.  `projectInformation` (商品情報サマリー、教育コンテンツサマリー、クリエイティブパーツなど)、`aiPersona` (AIの役割設定)、`userPersona` (読み込まれた話者ペルソナの内容)、`structurePattern` (読み込まれた構成パターンの内容)、`templateAndMaster` (クリエイティブパーツ要素マスター) をテンプレート内のプレースホルダーに挿入。
        *   **例**: `{{userPersona}}` に「情熱的開発者」のペルソナ定義が挿入される。`{{structurePattern}}` に「ショート動画の悩み共感・解決型」の構成パターンが挿入される。
    4.  必要に応じて `modificationInstruction` (修正指示) を追加。
    5.  AIがこのプロンプトと複数のナレッジを基に最終的な広告アウトプットを生成。
*   **ナレッジの役割**:
    *   `speaker_persona/*.md`: AIが特定のキャラクター（例: 情熱的開発者）になりきり、その口調、価値観、ターゲットへの影響を考慮して台本を生成するための詳細な指示を提供します。
    *   `structure/*.md`: 各広告フォーマット（例: ショート動画）における成功パターン（例: 悩み共感・解決型）の構成ステップと、それに使用すべきクリエイティブ要素（`creative_parts_master.md` で定義された要素）を具体的に指示し、再現性の高い広告作成を可能にします。

### 2.6. 追加処理プロンプト

*   **目的**: 生成された広告アウトプットに対するアノテーションや法的チェック。
*   **関連ファイル**:
    *   `src/lib/prompts/outputPrompts.ts`: `getAnnotationPromptTemplate` 関数と `getLegalCheckPromptTemplate` 関数。
    *   `src/lib/prompt_v2/ppt05_annotation.md`: アノテーション生成のプロンプトテンプレート。
    *   `src/lib/prompt_v2/ppt06.legal_check.md`: 法的チェックのプロンプトテンプレート。
*   **連携フロー**:
    1.  それぞれの関数が対応するMarkdownテンプレートを読み込む。
    2.  生成された広告アウトプットをインプットとして、AIが追加処理を実行。
*   **ナレッジの役割**: これらのMarkdownファイルは、アノテーションや法的チェックのプロセスと基準を定義するナレッジとして機能します。

## 3. アプリケーションコードでの利用

これらのプロンプトとナレッジの連携は、主に以下のアプリケーションコンポーネントとアクションによってオーケストレーションされます。

*   **UIコンポーネント**:
    *   `src/app/(project)/project/[projectId]/summary/SummaryClientPage.tsx`: 商品サマリーの表示・編集と、`generateProductSummaryPrompt` の呼び出し。
    *   `src/app/(project)/project/[projectId]/education/EducationClientPage.tsx`: 教育コンテンツの表示・編集と、`generateEducationContentPrompt` の呼び出し。
    *   `src/app/(project)/project/[projectId]/parts/CreativePartsClientPage.tsx`: クリエイティブパーツの表示・編集と、`generateCreativePartsPrompt` の呼び出し。
    *   `src/app/(project)/project/[projectId]/output/[outputId]/components/OutputPageClient.tsx`: 最終広告アウトプットの表示・編集と、`generateAdOutputPrompt` の呼び出し。
*   **サーバーアクション**:
    *   `src/app/actions/generationActions.ts`: 上記のプロンプト関数を呼び出し、AIモデルとの連携を通じてコンテンツ生成を実行するバックエンドロジック。
