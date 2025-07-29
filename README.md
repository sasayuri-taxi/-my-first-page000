<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>インタラクティブ同行援護研修プログラム (Gemini API搭載版)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Calm Harmony (Warm neutrals with slate and amber accents) -->
    <!-- Application Structure Plan: A responsive two-column layout is used. A sticky sidebar navigation allows users to easily jump between the 12 training sessions. The main content area is dynamically updated with JS, presenting one session at a time. This structure prevents overwhelming the user with a long scroll and allows for focused learning on a specific topic. Interactive elements like accordions for case studies and new Gemini API-powered generators encourage active engagement rather than passive reading. This design was chosen for its clarity, ease of navigation, and suitability for structured, interactive educational content. -->
    <!-- Visualization & Content Choices: Report Info: 12-month training plan -> Goal: Provide an easy-to-navigate overview and detailed exploration -> Viz/Method: A dynamic SPA with a sidebar for navigation and a main content area. Report Info: Case studies and solutions -> Goal: Encourage active thinking -> Viz/Method: JS-powered accordion/toggle to show/hide solutions. Report Info: Heinrich's Law (1:29:300 ratio) -> Goal: Visually represent the abstract ratio -> Viz/Method: A Chart.js bar chart to make the concept immediately understandable. Report Info: User-driven scenario generation -> Goal: Provide infinite, practical training scenarios -> Viz/Method: Gemini API call via a text input and button, displaying the generated text in a dedicated area. Report Info: User-driven description practice -> Goal: Allow users to practice a core skill -> Viz/Method: Gemini API call to generate model descriptions based on user input. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Noto Sans JP', 'Inter', sans-serif;
            background-color: #FDFBF8; /* Warm off-white */
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 400px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 350px;
            }
        }
        .content-card {
            background-color: #FFFFFF;
            border: 1px solid #E5E7EB;
            border-radius: 0.75rem;
            padding: 1.5rem;
            margin-bottom: 1.5rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
            transition: all 0.3s ease-in-out;
        }
        .nav-link {
            transition: all 0.2s ease-in-out;
        }
        .nav-link.active {
            background-color: #D97706; /* Amber 600 */
            color: white;
            transform: translateX(4px);
        }
        .solution-toggle {
            cursor: pointer;
            background-color: #FEF3C7; /* Amber 100 */
            color: #92400E; /* Amber 800 */
            transition: background-color 0.2s;
        }
        .solution-toggle:hover {
            background-color: #FDE68A; /* Amber 200 */
        }
        .solution-content {
            background-color: #FFFBEB; /* Amber 50 */
            border-left: 4px solid #F59E0B; /* Amber 500 */
        }
        .gemini-card {
            background-color: #F0F9FF; /* Sky 50 */
            border: 1px solid #BAE6FD; /* Sky 200 */
        }
        .gemini-button {
            background-color: #0284C7; /* Sky 600 */
            color: white;
            transition: background-color 0.2s;
        }
        .gemini-button:hover {
            background-color: #0369A1; /* Sky 700 */
        }
        .gemini-button:disabled {
            background-color: #7DD3FC; /* Sky 300 */
            cursor: not-allowed;
        }
        .loading-spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #0284C7;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="text-slate-800">

    <div class="max-w-screen-xl mx-auto p-4 md:p-8">
        <header class="text-center mb-10">
            <h1 class="text-3xl md:text-4xl font-bold text-slate-900">同行援護のプロを目指すための特別研修プログラム</h1>
            <p class="mt-4 text-lg text-slate-600">確かな技術と、一歩進んだ対応力をその手に (✨Gemini API搭載)</p>
            <p class="mt-2 text-md text-slate-500">監修：介護サービスセンターささゆり</p>
        </header>

        <div class="flex flex-col md:flex-row md:space-x-8">
            <aside class="w-full md:w-1/4 mb-8 md:mb-0">
                <div class="sticky top-8">
                    <h2 class="text-lg font-bold mb-4 text-slate-700">研修プログラム (全12回)</h2>
                    <nav id="session-nav" class="flex flex-col space-y-2"></nav>
                </div>
            </aside>

            <main id="session-content" class="w-full md:w-3/4">
            </main>
        </div>
    </div>

    <script>
        const trainingData = [
             {
                id: 0,
                title: "はじめに",
                icon: "🚀",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-4 text-slate-900">はじめに：頼れる支援者から、真のプロフェッショナルへ</h2>
                        <div class="space-y-6 text-slate-700">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">このプログラムが目指すもの</h3>
                                <p>この研修は、すでに同行援護の基本的な資格を持ち、現場での経験がある方を対象としています。これまでの研修が、サービス提供に必要な「基礎」と「応用」の技術を学ぶものだったのに対し、このプログラムは、さらにその先にある「プロとしての実践力」を身につけることを目指します。私たちの目標は、ただ手順通りに仕事ができる支援者を育てることではありません。複雑な状況でも的確に考え、判断し、周りを引っ張っていけるような「エキスパート」を育てることです。このプログラムでは、「何をするか」という知識だけでなく、「なぜそうするのか」「様々な状況にどう対応するか」という、現場で本当に役立つ知恵を学んでいきます。</p>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">2025年（令和7年）の新しい制度を見据えて</h3>
                                <p>このプログラムは、2025年4月から全国でスタートする同行援護の新しい研修制度を意識して作られています。この制度変更は、国全体として同行援護の質を高め、より専門的な人材を育てようというメッセージです。特に、サービス提供責任者になれる条件が変わり、一般課程を修了した人でも実務経験を積めば責任者になれるようになることは、皆さんにとって大きなキャリアアップのチャンスです。このプログラムを終えた皆さんは、新しい時代のニーズに応えるだけでなく、それを超える知識と技術を身につけ、これからの同行援護を引っ張っていくリーダーになることが期待されています。</p>
                            </div>
                             <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">1年で身につける3つの大切な力</h3>
                                <ol class="list-decimal list-inside space-y-1">
                                    <li><strong>物事を整理して分析する力：</strong> 複雑な状況（環境、人間関係、利用者の健康状態などが絡み合う場面）を冷静に分析し、問題解決の糸口を見つける力。</li>
                                    <li><strong>利用者が自分で考えて行動できるよう後押しする力：</strong> 支援者が何でもやってあげる「代行者」ではなく、利用者が自分で決め、自分の権利を主張し、便利なテクノロジーを使いこなせるように手助けする「案内役」になるための力。</li>
                                    <li><strong>自分の仕事ぶりを振り返り、次に活かす力：</strong> 自分の支援を客観的に見つめ直し、成功からも失敗からも学び、プロとして成長し続ける習慣。</li>
                                </ol>
                            </div>
                        </div>
                    </div>
                    <div class="content-card">
                        <h3 class="text-xl font-semibold mb-4 text-amber-700">一般的な資格研修との違い</h3>
                        <div class="overflow-x-auto">
                            <table class="w-full text-left border-collapse">
                                <thead>
                                    <tr>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">ポイント</th>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">一般的な資格研修（一般・応用課程）</th>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">この特別研修プログラム</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">一番の目標</td>
                                        <td class="p-3">サービスに必要な基礎知識・技術を身につける</td>
                                        <td class="p-3">難しいケースに対応し、リーダーシップをとる力をつける</td>
                                    </tr>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">法律の理解</td>
                                        <td class="p-3">同行援護の制度の概要を知る</td>
                                        <td class="p-3">障害者差別解消法を現場で活かし、配慮を求める交渉ができる</td>
                                    </tr>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">技術スキル</td>
                                        <td class="p-3">基本的な誘導の技術</td>
                                        <td class="p-3">いつもと違う環境での問題解決や、盲導犬の賢い判断方法を応用する</td>
                                    </tr>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">リスク管理</td>
                                        <td class="p-3">危険を察知し、緊急時に対応する</td>
                                        <td class="p-3">ヒヤリハットを分析し、事故を未然に防ぐ仕組みを作る</td>
                                    </tr>
                                    <tr>
                                        <td class="p-3 font-semibold">仕事の焦点</td>
                                        <td class="p-3">決められた仕事をきちんとこなす</td>
                                        <td class="p-3">利用者の「自分でやりたい」という気持ちを育て、応援する</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                `
            },
            {
                id: 1,
                title: "プロの仕事を支える法律と心構え",
                icon: "⚖️",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第1回：プロの仕事を支える法律と心構え</h2>
                        <p class="text-lg text-slate-600 mb-6">単にルールを知っているだけでなく、障害のある人の権利に関する法律を現場で活かし、プロとしての高い意識を持って行動できるようになる。</p>
                        
                        <div class="space-y-6">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">「障害者差別解消法」をしっかり理解する</h3>
                                <p>同行援護のプロは、サービスを提供するだけでなく、利用者の権利を守る大切な役割も担っています。そのためには、法律を深く理解することが欠かせません。この回では、「障害者差別解消法」の「不当な差別」と「合理的配慮」という2つの大切な考え方について、具体的な例を挙げてじっくり考えます。</p>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">ケーススタディ：お店や窓口で「配慮」をお願いする技術</h3>
                                <p class="mb-4">「合理的配慮」は、黙っていても提供されるものではなく、利用者からお願いし、相手と話し合う「交渉」によって実現することが多いです。プロの支援者には、この話し合いをスムーズに進める力が求められます。ここでは、様々な場面を想定したケーススタディで、具体的な解決策を学びます。</p>
                                <div class="space-y-4">
                                    ${createSolutionToggle(
                                        'シナリオ1：交通機関での解決策',
                                        '<strong>状況：</strong> 乗車したバスの運転手が、視覚障害のある利用者に同伴する車椅子利用者のためのスペース確保に非協力的です。',
                                        `<strong>解決策のポイント：</strong><br>冷静かつ論理的に、そして協力的な姿勢で対話を進めることが重要です。感情的にならず、運転手、利用者、そして他の乗客全員の安全を確保するという共通の目標に向かって話を進めます。<br>
                                        <ol class="list-decimal list-inside space-y-2 mt-2">
                                            <li><strong>まずは穏やかに、具体的なお願いをする：</strong>「お忙しいところすみません。車椅子の方が安全に乗車できるよう、こちらのスペースを空けていただくことは可能でしょうか？」と、丁寧にお願いします。</li>
                                            <li><strong>理由と必要性を明確に伝える：</strong>「車椅子を固定しないと、バスが揺れた際に本人が危険なだけでなく、周りのお客様にご迷惑をおかけする可能性もあります。安全のためにご協力をお願いします」と、安全という共通の利益に焦点を当てて話します。</li>
                                            <li><strong>具体的な代替案や行動を提案する：</strong>「もしよろしければ、『車椅子の方がご乗車されますので、この付近の席の移動にご協力ください』とアナウンスしていただけると、大変助かります」のように、こちらから具体的な行動を提案します。</li>
                                            <li><strong>法律の趣旨に触れる（最終手段として）：</strong>「法律で、交通機関には障害のある人への配慮が求められていると聞いております。安全な乗車にご協力いただけないでしょうか」と、冷静に伝えます。</li>
                                        </ol>`
                                    )}
                                    ${createSolutionToggle(
                                        'シナリオ2：小売店での解決策',
                                        '<strong>状況：</strong> 店員が多忙を理由に、商品のラベル読み上げを断ってきました。',
                                        `<strong>解決策のポイント：</strong><br>店員の「忙しい」という状況に理解を示しつつ、こちらのニーズも満たせるような、お互いの負担が少ない「落としどころ」を見つける交渉術が求められます。<br>
                                        <ol class="list-decimal list-inside space-y-2 mt-2">
                                            <li><strong>相手の状況に共感を示す：</strong>「お忙しいところ、本当に申し訳ありません」と、まず相手の状況への理解を示す一言を添えます。</li>
                                            <li><strong>お願いを具体的かつ最小限にする：</strong>「たくさんの商品をお願いするわけではなく、ただ、この商品の賞味期限だけ一言教えていただけないでしょうか？すぐに済みますので」と、お願いする内容を限定します。</li>
                                            <li><strong>選択肢を提案する：</strong>「もし今すぐが難しければ、少しこちらで待たせていただくか、他にお手すきの店員さんはいらっしゃいますでしょうか？」と尋ねます。</li>
                                            <li><strong>相手のメリットにも繋がることを示唆する：</strong>「こちらの商品情報を教えていただければ、すぐに購入を決められますので」と伝えることで、「対応すれば、結果的に自分の仕事が早く片付く」と相手に感じてもらうことができます。</li>
                                        </ol>`
                                    )}
                                    ${createSolutionToggle(
                                        'シナリオ3：行政手続きでの解決策',
                                        '<strong>状況：</strong> 行政窓口で、視覚障害のある利用者にはアクセスできない書類への記入を一方的に求められました。',
                                        `<strong>解決策のポイント：</strong><br>行政機関には合理的配慮を提供する「法的義務」があるため、その点を踏まえ、自信を持って、しかし協力的な態度で代替案を要求することが重要です。<br>
                                        <ol class="list-decimal list-inside space-y-2 mt-2">
                                            <li><strong>できない理由を明確に伝える：</strong>「ありがとうございます。ただ、こちらは視覚に障害があるため、この書類を自分で読んだり書いたりすることが難しい状況です」と、まずはっきりと伝えます。</li>
                                            <li><strong>具体的な配慮を法的な根拠と共に要求する：</strong>「恐れ入りますが、障害者差別解消法で定められている合理的配慮として、内容を読み上げていただき、本人の意思を確認しながら代筆していただくことは可能でしょうか？」と、具体的な代替案を法律上の根拠と共に提示します。</li>
                                            <li><strong>手続きの進め方をこちらから提案する：</strong>「私が隣で一つずつご本人に質問内容を伝え、ご本人が答えた内容を職員の方にお伝えしますので、それを書き込んでいただく、という形でお願いできますでしょうか」と、スムーズな進め方を具体的に提案します。</li>
                                            <li><strong>署名が必要な場合のサポート方法を伝える：</strong>「ご本人の署名が必要な箇所につきましては、枠の場所を指で示していただければ、私が手を添えてサポートいたします」と、署名についても適切な支援方法があることを伝えます。</li>
                                        </ol>`
                                    )}
                                </div>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">プロとしての高い倫理観を身につける</h3>
                                <p class="mb-4">基本的な研修でも職業倫理は学びますが、ここでは、現場で実際に判断に迷うような難しい場面に焦点を当てます。</p>
                                <div class="space-y-4">
                                    ${createSolutionToggle(
                                        '利用者との適切な距離感',
                                        '仕事としての支援と、個人的な親しさの境界線はどこにあるのか。長く関わる中で、どうやって適切な関係を保つか。',
                                        `<strong>解決策の考え方：</strong><br>目標は、信頼関係を築くための「親しみやすさ」と、プロとしての立場を守る「礼儀正しさ」のバランスをとることです。馴れ馴れしい態度は、かえって利用者に不快感を与え、信頼を損なう可能性があります。<br>
                                        <ol class="list-decimal list-inside space-y-2 mt-2">
                                            <li><strong>言葉遣いを基本にする：</strong>「です・ます」調を基本とし、柔らかい口調を心がけます。「お風呂行くよ！」ではなく、「そろそろお風呂の時間ですが、いかがですか？」のように、言葉を選ぶだけで印象は大きく変わります。</li>
                                            <li><strong>物理的な距離に配慮する：</strong>親しい関係でも、いきなり体に触れるのは避けます。介助で体に触れる前には「少し腕をお借りしますね」と一声かけることが大切です。</li>
                                            <li><strong>役割の境界線を守る：</strong>利用者からプライベートな相談を受けた場合、個人的に解決しようとせず、「それはお辛いですね。よろしければ、ケアマネージャーさんにも相談してみませんか？」と、適切な専門職につなぐのがプロの対応です。</li>
                                            <li><strong>事業所全体で対応を統一する：</strong>支援者によって距離感が異なると、利用者が特定の支援者に依存してしまう可能性があります。利用者との関わり方について、事業所全体で話し合い、方針を統一しておくことが大切です。</li>
                                        </ol>`
                                    )}
                                    ${createSolutionToggle(
                                        '秘密を守る義務',
                                        '病院への付き添いや冠婚葬祭への参加で知った、とてもプライベートな情報をどう扱うか。',
                                        `<strong>解決策の考え方：</strong><br>守秘義務は、介護・福祉の仕事における最も重要な原則の一つであり、法律でも定められています。違反した場合は罰則の対象にもなります。<br>
                                        <ol class="list-decimal list-inside space-y-2 mt-2">
                                            <li><strong>「秘密」の範囲を広く捉える：</strong>利用者の氏名や住所はもちろん、病歴、家族構成、経済状況、ケアプランの内容など、業務上知り得たすべての情報が対象です。</li>
                                            <li><strong>具体的なNG行動を知る：</strong>事業所の外で利用者の話をする、許可なくSNSに書き込む、家族や友人に話す、など。</li>
                                            <li><strong>例外規定を正しく理解する（最も重要）：</strong>利用者の生命や身体に重大な危険が迫っている場合や、虐待が疑われる場合は、秘密を守ることよりも、命や人権を守ることが優先されます。この場合、関係機関への通報は義務であり、守秘義務違反にはなりません。ただし、必ず事業所の責任者に報告し、組織として対応することが鉄則です。</li>
                                        </ol>`
                                    )}
                                    ${createSolutionToggle(
                                        '利用者の「やりたい」気持ちと安全のバランス',
                                        '利用者が選んだことが、客観的に見て少し危険かもしれない時、どう対応すべきか。「リスクを理解した上で挑戦する権利」を尊重しつつ、安全を確保するための話し合いと支援の方法を考えます。',
                                        `<strong>解決策の考え方：</strong><br>一方的に「ダメです」と禁止するのは、利用者の自己決定権を侵害する可能性があります。大切なのは、リスクを共有し、一緒に解決策を探すパートナーとしての姿勢です。<br>
                                        <ol class="list-decimal list-inside space-y-2 mt-2">
                                            <li><strong>ステップ1：リスクを客観的に評価する：</strong>まずは冷静に、その行動がどの程度の危険を伴うのかを評価します。</li>
                                            <li><strong>ステップ2：情報提供と対話：</strong>支援者として何が心配なのかを具体的に伝えます。「その道は人通りが少なく、何かあった時に心配です」のように、「私」を主語にして伝えます。</li>
                                            <li><strong>ステップ3：一緒に解決策を探る：</strong>「どうすれば安全にできるか、一緒に考えてみませんか？」と提案し、協力して問題解決にあたります。</li>
                                            <li><strong>ステップ4：代替案を提案する：</strong>どうしても危険な場合は、ただ禁止するのではなく、利用者の「やりたい」という根本的な気持ちを満たす別の方法を提案します。</li>
                                            <li><strong>ステップ5：記録と報告：</strong>話し合いの経緯や最終的な決定は必ず記録に残り、サービス提供責任者などに報告します。これにより、支援者一人で責任を抱え込むことを防ぎます。</li>
                                        </ol>`
                                    )}
                                </div>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 2,
                title: "目の病気と見え方の関係、そして「見えにくさ」へのケア",
                icon: "👁️",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第2回：目の病気と見え方の関係、そして「見えにくさ」へのケア</h2>
                        <p class="text-lg text-slate-600 mb-6">病気の名前と、それによって利用者が「実際にどう見えているか」を結びつけ、一人ひとりに合った具体的な支援方法を学べるようになる。</p>
                        <div class="space-y-6">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">病名から「見え方」と「支援方法」を考える</h3>
                                <p>同行援護のプロは、ただ「目の不自由な人」を支援するのではありません。「特定の病気によって、特有の見え方をしている一人ひとり」を支援します。この違いを理解することが、プロへの第一歩です。ここでは、主な病気が見え方にどう影響するかを深く学びます。</p>
                                <ul class="list-disc list-inside mt-2 space-y-1">
                                    <li><strong>緑内障：</strong>視野が中心に向かって狭くなる「トンネルビジョン」が特徴です。これが移動中にどんな危険（横から来る人や物にぶつかるなど）を生むかを考え、その危険を減らすための誘導方法や、利用者が周りを見渡すための声かけの仕方を学びます。</li>
                                    <li><strong>網膜色素変性症：</strong>暗いところで見えにくくなる「夜盲」と、視野がだんだん狭くなるのが特徴です。夜の外出や、レストランのような薄暗い場所でどう影響するかを考え、ペンライトの活用法や、段差の情報を具体的に伝えることの大切さを学びます。</li>
                                    <li><strong>糖尿病網膜症・黄斑変性症：</strong>見たい中心部分が見えない「中心暗点」や、物が歪んで見える「変視症」が、文字を読んだり、人の顔を認識したり、お金を扱ったりする時にどう影響するかを考えます。</li>
                                    <li><strong>脳の病気が原因の視覚障害：</strong>脳梗塞などが原因で起こる「同名半盲」について学びます。これは、左右両方の目の同じ側半分（例えば右半分）が見えなくなる状態で、片目が見えないのとは全く違います。この状態の人が、食事の時に皿の片側に気づかなかったり、廊下の壁にぶつかりやすかったりする理由を理解し、それに対応した支援方法を学びます。</li>
                                </ul>
                            </div>
                             <div class="overflow-x-auto">
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">病気ごとの「見え方」と「支援のヒント」早見表</h3>
                                <table class="w-full text-left border-collapse">
                                    <thead>
                                        <tr>
                                            <th class="border-b-2 border-slate-300 p-3 bg-slate-50">病名</th>
                                            <th class="border-b-2 border-slate-300 p-3 bg-slate-50">代表的な見え方</th>
                                            <th class="border-b-2 border-slate-300 p-3 bg-slate-50">困りごとの例</th>
                                            <th class="border-b-2 border-slate-300 p-3 bg-slate-50">支援のヒント</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <tr class="border-b border-slate-200">
                                            <td class="p-3 font-semibold">緑内障</td>
                                            <td class="p-3">視野が中心に向かって狭くなる（トンネルビジョン）</td>
                                            <td class="p-3">横から来る人や物に気づかずぶつかる。</td>
                                            <td class="p-3">支援者は利用者の少し斜め前を歩き、横からの危険を声で伝える。角を曲がる時は、大きく回るように誘導する。</td>
                                        </tr>
                                        <tr class="border-b border-slate-200">
                                            <td class="p-3 font-semibold">加齢黄斑変性</td>
                                            <td class="p-3">見たい中心が見えない（中心暗点）</td>
                                            <td class="p-3">メニューや人の顔が直接見えない。</td>
                                            <td class="p-3">メニューを利用者の視線の少し横に持っていく。人を説明する時は、顔ではなく服装や髪型、声で伝える。</td>
                                        </tr>
                                        <tr class="border-b border-slate-200">
                                            <td class="p-3 font-semibold">網膜色素変性症</td>
                                            <td class="p-3">暗い所で見えにくい、視野がだんだん狭くなる</td>
                                            <td class="p-3">薄暗い店や夜道で段差が見えない。</td>
                                            <td class="p-3">小さなライトで足元をそっと照らす。事前に段差の数や高さを具体的に伝える。夕方以降は明るい道を選ぶ。</td>
                                        </tr>
                                        <tr>
                                            <td class="p-3 font-semibold">白皮症（アルビニズム）</td>
                                            <td class="p-3">光にとても敏感（羞明）</td>
                                            <td class="p-3">日光や窓からの光がまぶしくて目を開けられない。</td>
                                            <td class="p-3">日陰の多い道を選ぶ。カフェなどでは窓を背にして座れる席に案内する。</td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 3,
                title: "情報提供の技術：言葉で風景を伝える達人になる",
                icon: "🗣️",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第3回：情報提供の技術：言葉で風景を伝える達人になる</h2>
                        <p class="text-lg text-slate-600 mb-6">言葉で情報を伝えるスキルを、単に事実を伝えるレベルから、まるで絵を描くように風景を伝えるレベルまで高める。</p>
                        <div class="space-y-6">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">美術館の解説から学ぶ「伝え方の基本」</h3>
                                <ul class="list-disc list-inside space-y-1">
                                    <li><strong>「全体から細かい部分へ」の原則：</strong> まずは広い視野で全体像を伝え、それから細かい部分を説明します。「両側に2階建てのビルが並ぶ、賑やかな商店街です」と全体を伝えた後、「あなたのすぐ右手に、青い日よけのパン屋さんがあります」と具体的に説明します。</li>
                                    <li><strong>自分の感想を入れず、見たままを伝える言葉遣い：</strong> 「きれいな公園ですね」のような主観的な表現ではなく、「中央に噴水があって、その周りをきれいな花壇と大きな木が囲んでいる、広々とした公園です」のように、客観的に描写します。</li>
                                    <li><strong>五感を使って伝える：</strong> 見たものだけでなく、音や匂い、手触りなども使って伝えます。「左のカフェから聞こえる食器の音」「コーヒーの香ばしい匂い」といった情報を加えることで、よりリアルで立体的なイメージを伝えることができます。</li>
                                </ul>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">情報を整理して、効果的に伝えるコツ</h3>
                                <ul class="list-disc list-inside space-y-1">
                                    <li><strong>利用者が「今、何を知りたいか」を一番に考える：</strong> 状況に応じて、伝える情報の優先順位を決めます。道を渡る時は、建物の色よりも信号の色や車の流れが最優先です。</li>
                                    <li><strong>一番大事なことから伝える：</strong> 「2歩先に、縁石があります」は、「縁石は、2歩先にあります」よりも、危険が伝わりやすく、すぐに行動につながります。</li>
                                    <li><strong>利用者が次に行動しやすくなるように伝える：</strong> 「あそこにベンチがあります」よりも、「あなたの2時の方向に、空いているベンチが一つあります」と伝える方が、利用者は「休憩する」という具体的な行動に移しやすくなります。</li>
                                </ul>
                            </div>
                            <div class="content-card gemini-card">
                                <h3 class="text-xl font-semibold mb-4 text-sky-800">✨ 状況描写アシスタント</h3>
                                <p class="mb-4 text-slate-600">練習したい場面のキーワード（例：「公園のベンチ」「交差点」）を入力して、AIに模範的な状況描写を作成させてみましょう。</p>
                                <div class="flex flex-col sm:flex-row gap-2">
                                    <input type="text" id="desc-input" class="flex-grow p-2 border border-slate-300 rounded-md" placeholder="例：夕方の商店街">
                                    <button id="generate-desc-btn" class="gemini-button font-bold py-2 px-4 rounded-md">描写を生成</button>
                                </div>
                                <div id="desc-result" class="mt-4 p-4 bg-white rounded-md min-h-[100px] border border-slate-200">
                                    ここに結果が表示されます。
                                </div>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 4,
                title: "複雑で予測不能な場所でのナビゲーション術",
                icon: "🗺️",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第4回：複雑で予測不能な場所でのナビゲーション術</h2>
                        <p class="text-lg text-slate-600 mb-6">基本的な誘導技術だけでは対応が難しい、いつもと違う、状況が変わりやすい場所で、安全かつスムーズに利用者を支援するための、高度な判断力と計画的に考える力を身につける。</p>
                        <div class="space-y-6">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">周囲の状況を分析し、先を読む力</h3>
                                <ul class="list-disc list-inside space-y-1">
                                    <li><strong>危険を見つけ、分解する：</strong> 工事中のデコボコ道、路上に置かれた看板、開いたままのマンホール、大きな駅の複雑な通路など、基本的な研修ではあまり扱わないような危険を体系的に見つけ、分析する訓練をします。</li>
                                    <li><strong>予測して計画を立てる：</strong> 音楽フェスやラッシュ時の駅のような複雑な場所に入る前に、頭の中でシミュレーション（メンタル・ウォークスルー）をします。起こりうる問題を予測し、メインの計画（プランA）だけでなく、予備の計画（プランB、プランC）も立てる練習をします。</li>
                                </ul>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">盲導犬の賢い判断に学ぶ</h3>
                                <p>私たちは盲導犬の代わりになることを目指すのではありません。盲導犬の高度な判断の仕方を学び、それを人間が応用することを目指します。</p>
                                <ul class="list-disc list-inside mt-2 space-y-1">
                                    <li><strong>賢い不服従（インテリジェント・ディスオビーディエンス）：</strong> 利用者の指示が危険な時に、あえて従わないという賢い判断です。例えば、利用者が「前へ」と指示しても、死角から静かに近づいてくる電気自動車がいる場合などです。</li>
                                    <li><strong>積極的な経路発見（プロアクティブ・パスファインディング）：</strong> 盲導犬はただ障害物をよけるだけでなく、一番安全で効率的な道を見つけます。私たちも、複雑な状況でいくつかの選択肢の中から、最も良い道を選ぶ練習をします。</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 5,
                title: "ケーススタディ：失敗と複雑な事例から学ぶ",
                icon: "📚",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第5回：ケーススタディ・ワークショップ：失敗と複雑な事例から学ぶ</h2>
                        <p class="text-lg text-slate-600 mb-6">実際にあった難しい事例や失敗談を体系的に分析することを通じて、失敗を責めるのではなく、そこから学ぶ姿勢を身につけ、継続的に支援の質を高めていく。</p>
                        <div class="space-y-6">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">事例の構造的な分析</h3>
                                <p class="mb-4">ここでは、単に「失敗談を共有する」だけでは終わりません。それぞれの事例について、決まった手順に沿って振り返り、分析します。</p>
                                <ul class="list-disc list-inside space-y-2">
                                    <li><strong>事例1：混雑した歩道での接触事故：</strong> なぜ事故は起きたのか？（狭い道、白杖を持っていなかったこと、支援者の思い込みなど）。どこで判断を間違えたか？どうすればリスクを減らせたか？</li>
                                    <li><strong>事例2：タクシーでの失禁トラブル：</strong> これは「計画と準備」の失敗です。外出前にどんな確認が足りなかったか？どんな予防策が考えられたか？（防水シートなど）。</li>
                                    <li><strong>事例3：コミュニケーションの失敗：</strong> 支援者が「あっち」のような曖昧な言葉を使ったり、何も言わずにそばを離れたりした。本当の原因は何か？技術不足か、配慮不足か。</li>
                                </ul>
                            </div>
                            <div class="content-card gemini-card">
                                <h3 class="text-xl font-semibold mb-4 text-sky-800">✨ AIシナリオジェネレーター</h3>
                                <p class="mb-4 text-slate-600">研修で使いたいシナリオのキーワード（例：「駅のホーム」「初めて行くレストラン」）を入力して、AIに新しいケーススタディを作成させてみましょう。</p>
                                <div class="flex flex-col sm:flex-row gap-2">
                                    <input type="text" id="scenario-input" class="flex-grow p-2 border border-slate-300 rounded-md" placeholder="例：大雨の日のバス停">
                                    <button id="generate-scenario-btn" class="gemini-button font-bold py-2 px-4 rounded-md">シナリオを生成</button>
                                </div>
                                <div id="scenario-result" class="mt-4 p-4 bg-white rounded-md min-h-[100px] border border-slate-200">
                                    ここに結果が表示されます。
                                </div>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">自分の仕事を振り返る習慣をつける</h3>
                                <p>参加者には、難しい仕事が終わった後に自分で振り返りを行うための簡単なフレームワーク（例：「うまくいったことは？」「難しかったことは？」「次にやるとしたら、どう変える？」）を紹介します。目標は、経験から学ぶことを習慣にすることです。</p>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 6,
                title: "生活の中の様々な場面での支援",
                icon: "🏦",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第6回：移動だけじゃない！生活の中の様々な場面での支援</h2>
                        <p class="text-lg text-slate-600 mb-6">専門的な知識が求められる場面や、プライベートで特に配慮が必要な場面で、利用者を効果的に支援するための知識と技術を身につける。</p>
                        <div class="grid md:grid-cols-2 gap-6">
                            <div class="bg-slate-50 p-4 rounded-lg">
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">役所や銀行での手続き</h3>
                                <p>契約書や公的な書類を正確に読み上げる「代読」、利用者の意思を完全に反映して記入する「代筆」のルール、専門用語を分かりやすく説明する役割を学びます。</p>
                            </div>
                             <div class="bg-slate-50 p-4 rounded-lg">
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">病院でのサポート</h3>
                                <p>自分の役割（医療行為はしない）をはっきりさせ、利用者が医師に症状を伝えたり、医師からの説明を理解したりするのを助けます。</p>
                            </div>
                             <div class="bg-slate-50 p-4 rounded-lg">
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">冠婚葬祭や会議への参加</h3>
                                <p>その場の雰囲気や人間関係を伝える「社会的」な支援を行います。誰がどこにいるか、誰が微笑んでいるかなどを伝え、利用者がその場の一員として楽しめるよう支援します。</p>
                            </div>
                             <div class="bg-slate-50 p-4 rounded-lg">
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">余暇活動への参加</h3>
                                <p>買い物支援に加え、地域のイベントや趣味の教室などへの参加を後押しします。目標は、単に「いる」だけでなく、「参加」して楽しめるようにすることです。</p>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 7,
                title: "盲ろう（目と耳の両方に障害がある）の方への支援入門",
                icon: "🤟",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第7回：盲ろう（目と耳の両方に障害がある）の方への支援入門</h2>
                        <p class="text-lg text-slate-600 mb-6">盲ろうの方たちの特別なニーズについて基本的な知識を学び、いつ専門の「盲ろう者向け通訳・介助員」にお願いすべきかを判断できるようになる。</p>
                        <div class="space-y-6">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">「盲ろう」の多様性を知る</h3>
                                <p>「盲ろう」は、単に「見えない＋聞こえない」ということではありません。その状態は、「全く見えず全く聞こえない」から「少し見えて少し聞こえる」まで様々です。また、生まれつきの方と、人生の途中でそうなった方とでは、コミュニケーションの方法や必要な支援が大きく異なります。</p>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">盲ろうの方とのコミュニケーションの基本</h3>
                                <p>この回は資格を取るためのコースではなく、あくまで「入門」です。基本的なコミュニケーション方法の理論を学び、どんな場面で使えるかを話し合います。</p>
                                <ul class="list-disc list-inside mt-2 space-y-1">
                                    <li><strong>触手話：</strong> 話し手の手話の形を、利用者が手で触って読み取る方法。</li>
                                    <li><strong>指点字：</strong> 点字の6つの点を、利用者の指に直接打ち込んで伝える方法。</li>
                                    <li><strong>手のひら書き：</strong> 利用者の手のひらに、指でひらがなやカタカナを書く方法。最初のコミュニケーション手段としてとても重要です。</li>
                                </ul>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">同行援護従業者の役割</h3>
                                <p class="font-bold text-red-700">この回の最も大切な結論は、「いつ専門家を呼ぶべきかを知る」ことです。</p>
                                <p>同行援護のプロの役割は、盲ろうの状況に気づき、基本的なコミュニケーションで安全と当面のニーズを確認し、その後、適切な専門サービス（盲ろう者向け通訳・介助員派遣事業など）につなぐことです。</p>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 8,
                title: "先を見越したリスク管理と事故防止の仕組みづくり",
                icon: "🛡️",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第8回：先を見越したリスク管理と事故防止の仕組みづくり</h2>
                        <p class="text-lg text-slate-600 mb-6">事故が起きてから対応する「後追い」の考え方から、リスクを予測し、根拠に基づいて分析し、事故を未然に防ぐ「先読み」の考え方へと転換する。</p>
                        <div class="space-y-8">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">ハインリッヒの法則（安全のピラミッド）</h3>
                                <p class="mb-4">1件の大きな事故の裏には、29件の小さな事故と、300件の「ヒヤリハット」（ヒヤリとしたりハッとしたりした出来事）が隠れている、という法則です。この300件のヒヤリハットをきちんと報告・分析し、その根本原因を取り除くことで、大きな事故そのものを防げます。</p>
                                <div class="chart-container">
                                    <canvas id="heinrichChart"></canvas>
                                </div>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">ケーススタディ：ヒヤリハットを分析する</h3>
                                <p class="mb-2">表面的な原因だけでなく、「なぜなぜ分析」のような手法を使って、より深い、仕組みとしての原因を探ります。</p>
                                <div class="bg-slate-50 p-4 rounded-lg border-l-4 border-slate-400">
                                    <p><strong>事例：利用者が道の段差でつまずきそうになった。</strong></p>
                                    <ul class="list-disc list-inside mt-2 space-y-1 text-sm">
                                        <li>なぜ？ 支援者が段差に気づかなかったから。</li>
                                        <li>なぜ？ スマホの地図を見ていて注意がそれていたから。</li>
                                        <li>なぜ？ 予定していた道が通れず、別の道を探していたから。</li>
                                        <li>なぜ？ 事前の計画で、その場所のGPSが不正確になる可能性を考えていなかったから。</li>
                                        <li>なぜ？ 事業所の外出計画のプロセスに、予備のルートを考える手順がなかったから。</li>
                                    </ul>
                                    <p class="mt-2 font-bold">本当の解決策：この場合、真の解決策は「支援者にスマホを見るなと注意する」ことではなく、「事業所の外出計画の仕組みを見直し、予備のルートを事前に準備することを必須にする」ことです。</p>
                                </div>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 9,
                title: "文化・芸術を楽しむお手伝い：美術館での体験",
                icon: "🎨",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第9回：文化・芸術を楽しむお手伝い：美術館での体験を通じた総まとめ</h2>
                        <p class="text-lg text-slate-600 mb-6">これまで学んだ複数のスキルを、美術館という管理されたリアルな状況を想定して、総合的に使う練習をする。</p>
                        <div class="space-y-6">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">第1段階：計画を立てる</h3>
                                <ul class="list-disc list-inside space-y-1">
                                    <li><strong>事前調査：</strong>美術館のウェブサイトなどを調べ、バリアフリーに関する情報を集める方法を学びます。</li>
                                    <li><strong>先を見越したコミュニケーション：</strong>美術館の担当部署に電話をかけ、特別なニーズについて事前に問い合わせる際のポイントを話し合います。</li>
                                    <li><strong>行動計画を立てる：</strong>事前調査と利用者の興味をもとに、訪問計画の案を作成します。</li>
                                </ul>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">第2段階：机上シミュレーション</h3>
                                <ul class="list-disc list-inside space-y-1">
                                    <li><strong>空間のナビゲーション：</strong>映像資料やシナリオをもとに、利用者を入り口から目的の展示室まで案内する際の言葉のかけ方や移動計画を考え、発表します。</li>
                                    <li><strong>描写の技術：</strong>第3回で学んだ高度な描写技術を使い、絵画や彫刻を説明します。大切なのは、一方的に説明するのではなく、会話をしながら一緒に楽しむ鑑賞方法（「対話型の美術鑑賞」）について議論することです。</li>
                                    <li><strong>触れるツールや多感覚ツールの活用：</strong>美術館に触れる模型がある場合や、自分で準備した場合を想定し、それらを言葉の説明と組み合わせて、利用者の理解を深める方法を議論します。</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                `
            },
            {
                id: 10,
                title: "災害への備えと緊急時の対応",
                icon: "🚨",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第10回：災害への備えと緊急時の対応</h2>
                        <p class="text-lg text-slate-600 mb-6">国の公式ガイドラインに基づき、大きな災害が起きる前から、起きた後まで、それぞれの段階で、視覚に障害のある利用者を効果的に支援するために不可欠な知識と判断力を身につける。</p>
                        <div class="overflow-x-auto">
                            <h3 class="text-xl font-semibold mb-2 text-amber-700">災害対応フェーズ別チェックリスト</h3>
                            <table class="w-full text-left border-collapse">
                                <thead>
                                    <tr>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">フェーズ</th>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">主な目標</th>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">支援者の優先行動</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">備え</td>
                                        <td class="p-3">利用者の準備を万全にする</td>
                                        <td class="p-3">非常用持ち出し袋の準備支援、地域防災訓練への参加を勧める、連絡手段の確認</td>
                                    </tr>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">発災直後</td>
                                        <td class="p-3">命の安全を確保する</td>
                                        <td class="p-3">すぐに身の安全を確保、素早い避難判断、必要なら強めの誘導</td>
                                    </tr>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">避難所到着</td>
                                        <td class="p-3">安全な拠点を作る</td>
                                        <td class="p-3">運営本部に状況を伝える、戦略的な場所（壁際、トイレ近く）を確保、トイレへの案内</td>
                                    </tr>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">避難所生活</td>
                                        <td class="p-3">健康と情報を守る</td>
                                        <td class="p-3">掲示物を口頭で伝える、配給や行列の支援、必要な配慮を要求する</td>
                                    </tr>
                                    <tr>
                                        <td class="p-3 font-semibold">復旧</td>
                                        <td class="p-3">安定した生活への移行</td>
                                        <td class="p-3">仮設住宅への移動支援、新しい環境での移動練習、長期的な支援への連携</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                `
            },
            {
                id: 11,
                title: "支援の未来：便利なテクノロジーの活用",
                icon: "📱",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第11回：支援の未来：便利なテクノロジー（支援技術）の活用</h2>
                        <p class="text-lg text-slate-600 mb-6">支援者が、現在そして未来の支援技術に関する実践的な知識を身につけ、これらのツールを利用者の生活にうまく取り入れるための「案内役」「先生」「トラブル解決役」としての役割を理解する。</p>
                        <div class="overflow-x-auto">
                            <h3 class="text-xl font-semibold mb-2 text-amber-700">支援技術の機能と応用マトリクス</h3>
                            <table class="w-full text-left border-collapse">
                                <thead>
                                    <tr>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">テクノロジー</th>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">主な機能</th>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">最適な利用場面</th>
                                        <th class="border-b-2 border-slate-300 p-3 bg-slate-50">支援者の役割</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">Seeing AI / Sullivan+</td>
                                        <td class="p-3">文字・物体認識</td>
                                        <td class="p-3">メニュー、郵便物、商品ラベルを自分で読む</td>
                                        <td class="p-3">カメラの向け方を教える、難しい文字は代わりに読む</td>
                                    </tr>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">Be My Eyes</td>
                                        <td class="p-3">遠隔での人的支援</td>
                                        <td class="p-3">「この牛乳の賞味期限は？」といった、とっさの視覚的な問題解決</td>
                                        <td class="p-3">明確な頼み方を教える、オフライン時のバックアップ</td>
                                    </tr>
                                    <tr class="border-b border-slate-200">
                                        <td class="p-3 font-semibold">Eye Navi</td>
                                        <td class="p-3">ナビゲーション</td>
                                        <td class="p-3">歩行ナビ、信号認識</td>
                                        <td class="p-3">事前にルートを計画する、機能しない時は自分がナビになる</td>
                                    </tr>
                                    <tr>
                                        <td class="p-3 font-semibold">AIスーツケース</td>
                                        <td class="p-3">自動での誘導</td>
                                        <td class="p-3">空港など、地図情報が整った大規模な屋内空間の移動</td>
                                        <td class="p-3">サービスの利用手続きを手伝う、地図範囲外で支援する</td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                `
            },
            {
                id: 12,
                title: "この仕事を長く続けていくために",
                icon: "❤️",
                content: `
                    <div class="content-card">
                        <h2 class="text-2xl font-bold mb-2 text-slate-900">第12回：この仕事を長く続けていくために：セルフケア、燃え尽き予防、そして仲間との支え合い</h2>
                        <p class="text-lg text-slate-600 mb-6">心と体の健康が、プロとしての大切な能力であることを理解し、セルフケア、燃え尽き症候群（バーンアウト）の予防、そして仲間同士の支え合いの仕組みを作るための方法を身につける。</p>
                        <div class="space-y-6">
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">「燃え尽き症候群（バーンアウト）」を理解する</h3>
                                <p>バーンアウトを個人の弱さではなく、ストレスの多い対人援助の仕事における、予測可能な職業上のリスクとして捉えます。同行援護の仕事に特有のストレス（常に気を張っている、感情をコントロールする、身体的な負担など）について話し合います。</p>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">仲間同士で支え合う仕組みを作る</h3>
                                <p>「ピア・スーパービジョン・グループ」という、仲間同士で相談し合うグループのモデルを紹介します。これは、上司がいない、秘密が守られる場で、支援者が難しいケースについて相談し、グループで解決策を考えたり、うまくいった方法や新しい情報を共有したりする場所です。</p>
                            </div>
                            <div>
                                <h3 class="text-xl font-semibold mb-2 text-amber-700">総まとめ：自分自身の成長のための計画（PDP）を作成する</h3>
                                <p>1年間のプログラムの締めくくりとして、各参加者は今後1～3年間の自分自身の成長計画（Personal Development Plan）を作成します。この計画では、さらに学びたい分野を決め、専門家としての目標を設定し、この仕事を長く続け、燃え尽きを防ぐための自分なりの戦略を考えます。</p>
                            </div>
                        </div>
                    </div>
                `
            }
        ];

        let currentSessionId = 0;
        let chartInstance = null;

        const sessionNav = document.getElementById('session-nav');
        const sessionContent = document.getElementById('session-content');

        function createSolutionToggle(title, situation, solution) {
            return `
                <div class="border border-slate-200 rounded-lg">
                    <div class="solution-toggle p-3 font-semibold rounded-t-lg flex justify-between items-center">
                        <span>${title}</span>
                        <span class="text-xl transform transition-transform duration-300">+</span>
                    </div>
                    <div class="solution-content p-4 hidden">
                        <p class="mb-2">${situation}</p>
                        <div class="text-slate-700">${solution}</div>
                    </div>
                </div>
            `;
        }
        
        async function callGemini(prompt, resultElement, buttonElement) {
            resultElement.innerHTML = '<div class="loading-spinner"></div>';
            buttonElement.disabled = true;

            const apiKey = ""; 
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
            
            const payload = {
                contents: [{
                    role: "user",
                    parts: [{ text: prompt }]
                }]
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`APIエラー: ${response.status} ${response.statusText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const text = result.candidates[0].content.parts[0].text;
                    resultElement.innerHTML = text.replace(/\n/g, '<br>');
                } else {
                    resultElement.innerText = 'AIからの有効な応答がありませんでした。';
                }
            } catch (error) {
                console.error("Gemini API Error:", error);
                resultElement.innerText = 'エラーが発生しました。コンソールで詳細を確認してください。';
            } finally {
                buttonElement.disabled = false;
            }
        }


        function renderNav() {
            sessionNav.innerHTML = '';
            trainingData.forEach(session => {
                const link = document.createElement('a');
                link.href = '#';
                link.innerHTML = `<span class="mr-2">${session.icon}</span> ${session.title}`;
                link.className = `nav-link block p-3 rounded-lg font-medium text-slate-700 hover:bg-amber-100 ${session.id === currentSessionId ? 'active' : ''}`;
                link.addEventListener('click', (e) => {
                    e.preventDefault();
                    currentSessionId = session.id;
                    renderNav();
                    renderContent();
                });
                sessionNav.appendChild(link);
            });
        }

        function renderContent() {
            const session = trainingData.find(s => s.id === currentSessionId);
            if (!session) return;

            sessionContent.innerHTML = session.content;

            if (chartInstance) {
                chartInstance.destroy();
                chartInstance = null;
            }

            if (session.id === 8) {
                renderHeinrichChart();
            }
            
            attachToggleListeners();
            attachGeminiListeners();
        }
        
        function attachToggleListeners() {
            document.querySelectorAll('.solution-toggle').forEach(toggle => {
                toggle.addEventListener('click', () => {
                    const content = toggle.nextElementSibling;
                    const icon = toggle.querySelector('span:last-child');
                    content.classList.toggle('hidden');
                    if (content.classList.contains('hidden')) {
                        icon.style.transform = 'rotate(0deg)';
                        icon.innerText = '+';
                    } else {
                        icon.style.transform = 'rotate(45deg)';
                        icon.innerText = '×';
                    }
                });
            });
        }
        
        function attachGeminiListeners() {
            const generateScenarioBtn = document.getElementById('generate-scenario-btn');
            if(generateScenarioBtn) {
                generateScenarioBtn.addEventListener('click', () => {
                    const input = document.getElementById('scenario-input');
                    const resultElement = document.getElementById('scenario-result');
                    if(input.value.trim() === '') {
                        resultElement.innerText = 'キーワードを入力してください。';
                        return;
                    }
                    const prompt = `あなたは同行援護の研修専門家です。以下のキーワードを元に、研修で使えるリアルなケーススタディのシナリオを一つ作成してください。状況、課題、考えるべき点を簡潔に、しかし具体的に記述してください。\n\nキーワード: 「${input.value}」`;
                    callGemini(prompt, resultElement, generateScenarioBtn);
                });
            }

            const generateDescBtn = document.getElementById('generate-desc-btn');
            if(generateDescBtn) {
                generateDescBtn.addEventListener('click', () => {
                    const input = document.getElementById('desc-input');
                    const resultElement = document.getElementById('desc-result');
                     if(input.value.trim() === '') {
                        resultElement.innerText = '場面を入力してください。';
                        return;
                    }
                    const prompt = `あなたは同行援護のプロで、言葉で風景を伝える達人です。以下の場面について、「全体から細かい部分へ」「客観的に」「五感を使って」という原則に基づき、視覚障害のある方に伝えるための状況描写を作成してください。\n\n場面: 「${input.value}」`;
                    callGemini(prompt, resultElement, generateDescBtn);
                });
            }
        }

        function renderHeinrichChart() {
            const ctx = document.getElementById('heinrichChart')?.getContext('2d');
            if (!ctx) return;

            chartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['重大な事故', '軽微な事故', 'ヒヤリハット'],
                    datasets: [{
                        label: '発生件数（比率）',
                        data: [1, 29, 300],
                        backgroundColor: [
                            'rgba(220, 38, 38, 0.6)',
                            'rgba(245, 158, 11, 0.6)',
                            'rgba(5, 150, 105, 0.6)'
                        ],
                        borderColor: [
                            'rgba(220, 38, 38, 1)',
                            'rgba(245, 158, 11, 1)',
                            'rgba(5, 150, 105, 1)'
                        ],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    indexAxis: 'y',
                    scales: {
                        x: {
                            beginAtZero: true,
                            type: 'logarithmic',
                            title: {
                                display: true,
                                text: '発生件数（対数スケール）'
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        title: {
                            display: true,
                            text: 'ハインリッヒの法則 (1:29:300)',
                            font: {
                                size: 16
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    return ` 比率: ${context.raw}`;
                                }
                            }
                        }
                    }
                }
            });
        }

        document.addEventListener('DOMContentLoaded', () => {
            renderNav();
            renderContent();
        });

    </script>
</body>
</html>

