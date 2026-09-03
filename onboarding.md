# 入职指南

本文档概述了我们在新协作者入职会议上告知他们的内容。

## 入职会议前一周

* 如果新协作者还不是 nodejs GitHub 组织的成员，请确认他们正在使用[双因素身份验证][]。如果他们没有使用双因素身份验证，则无法将他们添加到组织中。请他们使用安全的双因素方法：身份验证器应用、通行密钥、安全密钥和/或 GitHub 移动应用。
* 建议新协作者安装 [`@node-core/utils`][] 并[为其设置凭据][]。

## 入职会议前十五分钟

* 在入职会议之前，将新协作者添加到 [协作者团队](https://github.com/orgs/nodejs/teams/collaborators)。
* 询问他们是否想加入任何[子系统团队](https://github.com/orgs/nodejs/teams/core/teams)，并相应地将他们添加进去。请参阅[议题追踪器中应该抄送谁][who-to-cc]。

## 入职会议

* 本次会议将涵盖：
  * [本地设置](#本地设置)
  * [项目目标和价值观](#项目目标和价值观)
  * [管理议题追踪器](#管理议题追踪器)
  * [审查拉取请求](#审查拉取请求)
  * [落地拉取请求](#落地拉取请求)

## 本地设置

* git：
  * 确保你有 `whitespace=fix`：`git config --global --add apply.whitespace fix`
  * 始终在你自己的 GitHub 复刻中为拉取请求创建一个分支
    * `nodejs/node` 仓库中的分支仅用于发布线
  * 将规范的 nodejs 仓库添加为 `upstream` 远程仓库：
    * `git remote add upstream git@github.com:nodejs/node.git`
  * 从 `upstream` 更新：
    * `git checkout main`
    * `git fetch upstream HEAD`
    * `git reset --hard FETCH_HEAD`
  * 为你提交的每个拉取请求创建一个新分支。
  * 成员资格：考虑将你在 Node.js GitHub 组织中的成员资格设为公开。这有助于识别协作者。如何操作的说明请参见 [公开或隐藏组织成员身份][]。

* 通知：
  * 使用 <https://github.com/notifications> 或设置电子邮件
  * 监视主仓库会塞满你的收件箱（通常工作日会有数百条通知），所以请做好准备
  * 建议监视[协作者仓库](https://github.com/nodejs/collaborators)中的讨论。

项目有一个实时讨论的场所：

* [OpenJS Foundation Slack](https://slack-invite.openjsf.org/) 上的 [`#nodejs-core`](https://openjs-foundation.slack.com/archives/C019Y2T6STH)

## 项目目标和价值观

* 协作者是项目的集体所有者
  * 项目有其贡献者的目标

* 有一些更高层次的目标和价值观
  * 对用户的同理心很重要（这也是我们为何要让新人入职的部分原因）
  * 总体而言：尽量对他人友善！
  * 最好的结果是，那些来到我们议题追踪器的人觉得他们可以再次回来。

* 你应该遵守 _并且_ 要求他人遵守[行为准则][]。

## 管理议题追踪器

* 你（基本上）拥有自由裁量权；如果你确信某个议题应该关闭，请不要犹豫，直接关闭。
  * 关闭议题时要友善！让人们知道原因，并说明如果有必要，议题和拉取请求可以重新打开。

* 请参阅[标签][]。
  * 有一个[机器人](https://github.com/nodejs-github-bot/github-bot) 会自动应用子系统标签（例如 `doc`、`test`、`assert` 或 `buffer`），以便我们了解拉取请求修改了代码库的哪些部分。当然，它并不完美。请随时为拉取请求和议题应用相关标签并移除无关标签。
  * `semver-{minor,major}`：
    * 如果某个更改有 _可能_ 破坏某些东西，请使用 `semver-major` 标签
    * 当添加 `semver-*` 标签时，请添加一条评论解释你为什么要添加它。立即执行，以免忘记！
  * 如果适用，请为拉取请求添加 [`author-ready`][] 标签。

* 请参阅[议题追踪器中应该抄送谁][who-to-cc]。
  * 随着时间的推移，这会变得更加自然
  * 对于那里列出的许多团队，如果你感兴趣，可以要求加入
    * 有些是工作组，添加人员有一定的流程，其他仅用于通知

* 当讨论变得激烈时，你可以通过在私密的 [nodejs/moderation](https://github.com/nodejs/moderation) 仓库中创建议题，请求其他协作者关注此事。请注意，虽然该仓库不是公开的，但 nodejs 组织中的任何人都可以访问，因此请勿将其用于举报个人（当然，在那里举报垃圾邮件/机器人是可以的）。
  * 这是一个 `nodejs` GitHub 组织的所有成员（不仅仅是 Node.js 核心的协作者）都可以访问的仓库。其内容不应外泄。
  * Node.js 有一个审核团队，当你不确定是否要在 Node.js 组织中采取行动时，可以联系他们。
  * 你可以自行审核非协作者的帖子。请根据审核政策报告所采取的审核行动。
  * 你始终可以参考[完整的审核政策](https://github.com/nodejs/admin/blob/main/Moderation-Policy.md)。
  * 你可以联系[审核团队成员完整列表](https://github.com/nodejs/admin/blob/main/Moderation-Policy.md#current-members-of-moderation-team)中的某个人。

## 审查拉取请求

* 首要目标是让代码库变得更好。

* 次要目标（但也相距不远）是让提交代码的人取得成功。来自新贡献者的拉取请求是发展社区的机会。

* 一次审查一点。不要压垮新贡献者。
  * 很容易陷入微观优化。不要屈从于那种诱惑。我们经常更改 V8。今天能提供更好性能的技术，将来可能不再需要。

* 请注意：你的意见分量很重！

* 小问题（对非必要的微小更改的请求）是可以的，但尽量避免拖延拉取请求。
  * 在评论时将它们标记为小问题：`Nit: change foo() to bar().`
  * 如果它们正在拖延拉取请求，请在合并时自行修复。

* 在可能的范围内，问题应由工具而非人工审阅者来识别。如果你留下关于本可由工具识别但尚未识别的问题的评论，请考虑实现必要的工具。

* 评论等待的最短时间
  * 对于非平凡的更改，我们尽量遵守最短等待时间，以便在这个分布式项目中可能有重要意见的人能够回应。
  * 对于非平凡的更改，请将拉取请求保持开放至少 48 小时。
  * 如果拉取请求被放弃，请检查他们是否介意你接手（特别是如果只剩下一些小问题）。

* 批准更改
  * 协作者使用 GitHub 的批准界面表示他们已审查并批准拉取请求中的更改
  * 有些人喜欢评论 `LGTM`（“Looks Good To Me”）
  * 你有权批准任何其他协作者的工作。
  * 你不能批准自己的拉取请求。
  * 当明确使用 `Changes requested` 时，请展现同理心——即使你不使用它，评论通常也会得到处理。
    * 如果你使用了，最好稍后能检查你的评论是否已得到处理
    * 如果你看到请求的更改已做出，你可以清除另一位协作者的 `Changes requested` 审查。
    * 使用 `Changes requested` 表示你认为某些评论会阻止拉取请求落地。

* 哪些属于 Node.js：
  * 意见不一——正因如此，拥有广泛的协作者基础是件好事！
  * 如果 Node.js 本身需要它（由于历史原因），那么它就属于 Node.js。
    * 也就是说，`url` 因为 `http` 而在那里，`freelist` 因为 `http` 而在那里，等等。
  * 那些无法在核心之外完成的事情，或者只有付出巨大代价才能完成的事情，例如 `async_hooks`。

* 持续集成（CI）测试：
  * <https://ci.nodejs.org/>
    * 它不是自动运行的。你需要手动启动。
  * CI 上的登录与 GitHub 集成。现在尝试登录吧！
  * 大多数时候你会使用 `node-test-pull-request`。现在就去那里吧！
    * 考虑将其加入书签：<https://ci.nodejs.org/job/node-test-pull-request/>
  * 要进入启动任务的表单，请点击 `Build with Parameters`。（如果你没有看到它，那可能意味着你没有登录！）现在就点击它吧！
  * 要从这个屏幕启动 CI 测试，你需要在表单中填写两个元素：
    * 应勾选 `CERTIFY_SAFE` 框。勾选它表示你已审查了要测试的代码，并且确信它不包含任何恶意代码。（例如，我们不希望有人劫持我们的 CI 主机来攻击互联网上的其他主机！）
    * `PR_ID` 框应填入标识包含你要测试的代码的拉取请求的编号。例如，如果拉取请求的 URL 是 `https://github.com/nodejs/node/issues/7006`，则在 `PR_ID` 中填入 `7006`。
    * 表单上的其余元素通常保持不变。
  * 如果你在 CI 相关方面需要帮助：
    * 使用 [Build WG 仓库](https://github.com/nodejs/build) 向维护 CI 基础设施的 Build WG 成员提交议题。

## 落地拉取请求

请参阅协作者指南：[落地拉取请求][]。

属于同一逻辑更改的一个拉取请求中的提交应被压缩。在入职练习中很少出现这种情况，因此需要在入职期间单独指出。

<!-- TODO(joyeechueng): 提供关于“一个逻辑更改”的示例 -->

## 练习：提交一个将你自己添加到 README 中的拉取请求

* 示例：
  <https://github.com/nodejs/node/commit/6669b3857f0f43ee0296eb7ac45086cd907b9e94>
  * 原始提交信息：
    `git show --format=%B 6669b3857f0f43ee0296eb7ac45086cd907b9e94`
* 协作者按 GitHub 用户名的字母顺序排列。
* 可选择包含你的个人代词。
* 提交，包括一个 `Fixes: <collaborator-nomination-issue-url>` 尾部，这样当提交落地时，提名议题 URL 将被自动关闭。
* 运行 `tools/find-inactive-collaborators.mjs`。如果该命令输出你的名字，请修改提交以包含对 [mailmap](.mailmap) 文件的添加。有关 mailmap 文件格式的信息，请参阅 [gitmailmap](https://git-scm.com/docs/gitmailmap)。
* 将提交推送到你自己的复刻。
* 为你的拉取请求打上 `doc`、`notable-change` 和 `fast-track` 标签。`fast-track` 标签应使 Node.js GitHub 机器人在拉取请求中发布一条评论，要求协作者通过在评论上留下 👍 反应来批准该拉取请求。
* 可选：在拉取请求上运行 Jenkins CI。使用 [`node-test-pull-request`][] 任务。为方便起见，你可以对拉取请求应用 `request-ci` 标签，让一个 GitHub Actions 工作流为你启动 Jenkins CI 任务。
* 在获得两名协作者对更改的批准和两名协作者对快速跟踪的批准后，落地该 PR。如果你已启动完整的 Jenkins CI，请从 Jenkins UI 取消它，因为该 PR 仅涉及文档更改，不需要完整的 CI 运行，它只是作为练习而运行。
* 如果在合理时间内没有足够的批准，请将入职 TSC 成员的单次批准视为足够，并落地该拉取请求。
  * 请务必添加 `PR-URL: <full-pr-url>` 和适当的 `Reviewed-By:` 元数据。
  * [`@node-core/utils`][] 自动化了元数据的生成和落地过程。请参阅 [`git-node`][] 的文档。
  * [`core-validate-commit`][] 自动化了提交信息的验证。这将在 `git node land --final` 命令期间运行。
  * 通常你可以直接使用 `commit-queue` 标签，让 Node.js GitHub 机器人将提交排队等待落地。但作为练习，学习如何手动落地提交也很有用，以防机器人或 CI 出现故障。
* 如果你手动落地提交，为了让它在 GitHub 上显示为“已合并”，在你在本地 `main` 分支上准备好落地的提交后，运行：

  ```bash
  git push --force-with-lease your-fork-remote HEAD:your-pr-branch # 更新你复刻中的 PR 分支。
  git push upstream main  # 将落地的提交推送到 upstream main 分支。
  ```

  GitHub 将自动检测到 PR 分支现在与 `main` 分支相同，并将 PR 标记为“已合并”。

## 最后说明

* 不要担心犯错：每个人都会犯错，有很多东西需要内化，这需要时间（我们认识到这一点！）
* 你几乎犯的任何错误都可以被修复或还原。
* 现有的协作者信任你并感谢你的帮助！
* 其他仓库：
  * <https://github.com/nodejs/TSC>：治理讨论和 TSC 投票
  * <https://github.com/nodejs/build>：构建基础设施讨论和 CI 问题
  * <https://github.com/nodejs/nodejs.org>：Node.js 网站和博客
  * <https://github.com/nodejs/Release>：发布管理和发布计划
  * <https://github.com/nodejs/citgm>：用于针对 Node.js 更改测试流行包的工具
  * <https://github.com/nodejs/admin>：管理问题和更改 Node.js GitHub 组织的请求（例如创建新仓库、新团队、添加组织范围的令牌）。
  * <https://github.com/nodejs/moderation>：审核评论或阻止垃圾邮件发送者的请求。
* OpenJS 基金会定期为 Node.js 项目的活跃贡献者举办峰会，我们会就项目工作进行面对面的讨论。基金会有差旅基金来支付[参与者的费用][]，包括住宿、交通和签证费（即使签证被拒），如果需要的话。有关详细信息，请查看 [summit](https://github.com/nodejs/summit) 仓库。
* 如果你有兴趣帮助修复 coverity 报告，请考虑按照[静态分析][]中的说明请求访问项目的 coverity 项目。
* 如果你有兴趣帮助提高 CI 可靠性，请查看[可靠性仓库][]和[如何处理 CI 不稳定性指南][]。在修复不稳定的测试时，建议在 main 分支和测试分支上运行 [`node-stress-single-test`][]，以验证修复能使测试在重复运行下更加稳定。

[行为准则]: https://github.com/nodejs/admin/blob/HEAD/CODE_OF_CONDUCT.md
[标签]: doc/contributing/collaborator-guide.md#labels
[落地拉取请求]: doc/contributing/collaborator-guide.md#landing-pull-requests
[公开或隐藏组织成员身份]: https://help.github.com/articles/publicizing-or-hiding-organization-membership/
[`@node-core/utils`]: https://github.com/nodejs/node-core-utils
[`author-ready`]: doc/contributing/collaborator-guide.md#author-ready-pull-requests
[`core-validate-commit`]: https://github.com/nodejs/core-validate-commit
[`git-node`]: https://github.com/nodejs/node-core-utils/blob/HEAD/docs/git-node.md
[`node-stress-single-test`]: https://ci.nodejs.org/job/node-stress-single-test/
[`node-test-pull-request`]: https://ci.nodejs.org/job/node-test-pull-request/
[如何处理 CI 不稳定性指南]: https://github.com/nodejs/test?tab=readme-ov-file#protocols-in-improving-ci-reliability
[参与者的费用]: https://github.com/openjs-foundation/cross-project-council/blob/main/community-fund/COMMUNITY_FUND_POLICY.md#community-fund-rules
[可靠性仓库]: https://github.com/nodejs/reliability
[设置凭据]: https://github.com/nodejs/node-core-utils#setting-up-github-credentials
[静态分析]: doc/contributing/static-analysis.md
[双因素身份验证]: https://help.github.com/articles/securing-your-account-with-two-factor-authentication-2fa/
[who-to-cc]: doc/contributing/collaborator-guide.md#who-to-cc-in-the-issue-tracker