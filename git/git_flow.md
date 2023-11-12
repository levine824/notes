# 前言

没有哪一种 Git 工作流是万能的，合适的 Git 工作流往往取决于项目的代码规模、协作人数、应用场景等，如：适用于小型 Monorepo 的 Feature Branch 工作流，适用于中大型 Monorepo 的 Trunk-based 工作流，本文会给出一些选型标准供打架参考，希望大家能找到合适自己 Monorepo 工程的 Git 工作流！

企业的研发团队在研发产品功能时通常会选择 2 种分支管理策略，它们分别是 Feature Branch Development 和 Trunk-based Development。2 种分支管理策略都有它们适用的场景，比如：在 github 上研发开源软件时，经常会使用 Feature Branch Development 模式，而 Google，Facebook，LinkedIn，微软常常会使用 Trunk-based Development 模式。企业在实施 CI（持续集成）时通常需要 Trunk-based Development 方面的实践，原因在于这种模式能够快速输出集成的结果。

我们最熟悉的 Git 工作流莫过于 Git flow、Gilab flow 和 Github flow，而对于 Feature Branch 和 Trunk-Based 比较陌生，那么以上几种 flow 有什么关系呢？

1. Feature Branch 和 Trunk-based 工作流是比较新晋的概念，二者是相对的、互斥的，它们组成一个全集；
2. Git flow、Gilab flow 和 Github flow 都属于 Feature Branch Development，它们有一个共同点：都采用『功能驱动式开发』，即：需求是开发的起点，先有需求再有功能分支或者补丁分支。

![git_flow_1](..\assets\git_flow_1.png)

# 适用场景

在 Monorepo 工程中，使用 Feature Branch Development 开发模式时，随着代码库复杂性和团队规模的增长，并行的『长期分支』也会越来越多，这些分支在合入主干时，将会频繁遇到冲突或者不兼容的情况，而手动解决代码冲突往往会引入 Bug。

而 Trunk-Based Development 鼓励开发者可以通过一些小的提交创建『短期分支』，从而大大缓解冲突问题，有助于保持生产版本的流畅。

**总的来说，选择一个工作流不仅仅是一系列操作工具的流程，我们往往还需要对它背后的思想买单；下面的表格是两种工作流模式在各个维度的适用情况：**

![git_flow_2](..\assets\git_flow_2.png)

# Feature Branch Development

## 什么是 Feature Branch Development

定义：『功能分支开发模式』的核心思想是所有特性开发都应该在专用的分支，而不是主分支中进行。这种封装使多个开发人员，可以轻松地在不干扰主代码库的情况下处理特定功能。这也意味着主分支永远不会包含损坏的代码，这对于持续集成环境来说是一个巨大的优势。

以下简单介绍下常用的几种 Feature Branch Development。

## Git Flow

![git_flow_3](..\assets\git_flow_3.png)

Git 的分支主要分为两个长期分支 master 、develop ；三个短期分支 hotfix 、release 和 feature。

**分支介绍：**

两个长期分支：

- master 分支：主干分支，用于存储正式发布的产品，任何时候在这个分支获取到的都是稳定的已发布的版本，与线上版本同步；master 分支只能通过与其他分支合并来更新内容，禁止直接在 master 分支进行修改；
- develop 分支：开发主分支，用于日常开发，包括代码优化、功能性开发等，是进行任何新的开发的基础分支，该分支汇集所有已经完成的功能，通过与其他分支合并来更新内容，并等待被整合到 master 分支中。

三个短期分支：

- feature 分支：功能开发分支，可以存在多个，用于开发功能，每一个新的功能可以创建一个新的分支；
- hotfix 分支：补丁分支，用来修复生产环境中的紧急 bug，每个线上的 bug 都需要从 master 分支上创建一个对应的 hotfix 分支；
- release 分支：准备要发布版本的分支，当 develop 版本的代码处于成熟时，基于 develop 创建一个 release 分支。

**分支管理：**

1. master 分支

   操作要求：该分支只有开发 leader 具有 merge 权限，不允许 commit/push，仅允许来自 release/、 hotfix/ 分支的合并，每次接受合并后创建 TAG；

   对应环境：生产环境；

   Tag 命名规范：master 分支只有 1 个，名称即为 master，代码合并到 master 需打Tag，示例：vA.B.C：A 为大版本号，B 为小版本号，C 为补丁版本号；

   生命周期：伴随项目的整个生命周期过程；

2. develop 分支

   操作要求：该分支只有开发 leader 具有 merge 权限，不允许 commit/push，从 master 分支克隆，允许来自 feature/ 、release/、hotfix/ 分支的合并；

   对应环境：开发环境；

   Tag 命名规范：develop 分支只有 1 个，名称即为 develop；

   生命周期：开始开发后一直存在；

3. feature 分支

   操作要求：允许 commit/push，从 develop 分支克隆, 并在该 feature 分支上进行开发，功能开发完合并到  develop 分支，最后删除该分支；

   对应环境：本地环境；

   分支命名规范：feature/vA.B.0/，A 为大版本号，B 为小版本号，为功能简述，例：feature/v1.0.0/order-detail；

   生命周期：当前 feature 开发期间；

4. hotfix 分支

   操作要求：允许 commit/push，从 master 分支克隆，并在该分支上进行 bug 修复，修复完成后，将 hotfix/ 分支合并到 master 分支和 develop 分支，最后删除该 hotfix/ 分支；

   对应环境：补丁环境；

   分支命名规范：hotfix/vA.B.C，A 为大版本号，B 为小版本号，C 为补丁版本号，C≠０，例：hotfix/v1.0.1；

   生命周期：确认要修复 Bug 到重新发布到 master 并打tag以及合并到 develop 分支；

5. release 分支

   操作要求：允许 commit/push，当 develop 分支上的项目准备发布时，从 develop 分支上创建一个新的 release 分支，发布任务时，需要将 release 分支合并到 master 分支完成发布，然后再将 release/ 分支代码合并到 develop 分支，以保证代码的一致性，最后删除 release 分支；

   对应环境：测试环境；

   分支命名规范：release/vA.B.0，A 为大版本号，B 为小版本号，例：release/v1.1.0；

   生命周期：版本功能开发完到发布并打 tag，合并到 master 及 develop 分支；

## Github Flow

GitHubFlow 来源于 GitHub 团队的工作实践。当代码托管在 GitHub 上时，则需要使用 GitHub Flow。相比 Git Flow 而言，GitHub Flow没有那么多分支。

GitHub Flow 通常只有一个 Master 分支是固定的，而且 GitHub Flow中的 Master 分支通常是受保护的，只有特定权限的人才可以向Master 分支合入代码。

在 GitHub Flow中，新功能开发或修复 Bug 需要从 Master 分支拉取一个新分支，在这个新分支上进行代码提交；功能开发完成，开发者创建 Pull Request（简称 PR），通知源仓库开发者进行代码修改 review，确认无误后，将由源仓库开发人员将代码合入 Master 分支。

GitHub Flow 只有 master 和 feature 分支，相当于把 master 和 develop 分支的作用进行了合并，模型图如下：

![git_flow_4](..\assets\git_flow_4.png)

GitHub Flow 简化了分支的管理，转而使用 PR（Pull request）的方式解决代码的合并。

如果需要开发一个新的 feature，在 master 分支的基础上创建 feature 分支，新的功能开发完成后提交一个 PR，再由其它的 review 人员对这个 PR 进行 review、测试、评审等；通过后将这个 PR 合并到 master 并删除 feature 分支。

## Gitlab Flow

GitLabFlow 出现的最晚，GitLabFlow 是开源工具 GitLab 推荐的做法。

GitLabFlow 支持 GitFlow 的分支策略，也支持 GitHubFlow 的“Pull Request”（在 GitLabFlow 中被称为“Merge Request”）。

相比于 GitHubFlow，GitLabFlow 增加了对预生产环境和生产环境的管理，即 Master 分支对应为开发环境的分支，预生产和生产环境由其他分支（如：Pre-Production、Production）进行管理。在这种情况下，Master 分支是 Pre-Production 分支的上游，Pre-Production 是 Production 分支的上游；GitLabFlow 规定代码必须从上游向下游发展，即新功能或修复 Bug 时，特性分支的代码测试无误后，必须先合入 Master 分支，然后才能由 Master 分支向 Pre-Production 环境合入，最后由 Pre-Production 合入到 Production。

GitLabFlow 中的 Merge Request 是将一个分支合入到另一个分支的请求，通过 Merge Request 可以对比合入分支和被合入分支的差异，也可以做代码的 Review。

# Trunk-Based Development

## 什么是 Trunk-based Development

Trunk-based Development：所有研发人员围绕主分支 trunk (也就是 github 上的 master 分支) 来共同研发，在研发过程中拒绝创建存活时间较长的分支，并使用 Feature Toggles（功能切换，如：功能开关） 和 Branch by Abstraction（抽象分支，代码重构可以使用，如：hibernate 切换 mybatis） 等技术在主分支上逐步发布需要长时间（通常是1周）才能研发完成的功能。

Trunk-based Development 的目的在于解决合并和持续构建的问题。下图展示了采用 Trunk-based development 来进行软件研发时所涉及的一系列活动：

![git_flow_5](..\assets\git_flow_5.png)

上图规定了以下规则：

1. 所有研发人员直接在 trunk上 提交代码；
2. 对外发布产品的时候需要从 trunk 上拉取 release 分支（比如：1.1.x 和 1.2.x），并基于 release 分支来发布(比如：1.1.0 和 1.1.1)；
3. release 分支中出现的 bug 或者需要性能优化时，则需要在 trunk 上完成，并通过 cherry-pick（`git cherry-pick <commitHash>`） 的方式在 trunk 中挑选对应的 commits 合并到 release 分支，此时的小版本号从 1.1.0 变成1.1.1‘；
4. 对外发布新功能时，需要基于 trunk 分支，重新拉取 release 分支，版本号从 1.1.x 变成 1.2.x，同时 1.1.x 的 release 分支将被废弃；
5. 在 trunk 分支上，每个 commit 之间的提交间隔很短，通常在一天之内提交好几次，甚至更多次。

在这些规则之下，研发者可以持续地在 trunk 分支中提交代码，而且期间没有合并。为了能够使得每一次提交都能够顺利地在 trunk 分支中通过，则需要一些技巧来实现 CI。

## 需要掌握哪些技巧来实践 Trunk-based Development

为了使 Trunk-based Development 能够在研发团队中顺利开展起来，需要团队成员掌握以下技巧，并且达成共识。

1. 将任务划分成许多可以在 1 天以内完成的小模块

快速验证想法的第一步就是将一个任务分解成多个可测试的子任务，并逐步实现。完成这些子任务所需的时间不应该超过一天，这么做的原因是每次完成子任务所需要的改动影响范围较小，而且这些改动被提交之后，其他团队成员能够及时看到，从而使得团队作为整体，清楚产品的研发状况。

2. 针对研发的功能编写自动化测试用例，并在本地验证

团队中的每一名研发人员都应该针对自己的研发任务来编写对应的单元测试，并在研发结束之后，在本地运行这些单元测试来验证正在研发的功能。除此之外，在开始研发时需要从 trunk 获取最新代码，并在本地运行已有的单元测试，确保拿到的代码是正常的。在设计单元测试时，须遵守的原则是每一个单元测试能够独立运行并且能够在短时间内运行结束（通常在本地执行所有单元测试所需的时间不应该超过 5 分钟）。在获取或提交代码时，在本地运行单元测试的原因是确保拿到的或者即将提交的代码能够正常工作，从而降低了破坏 trunk的风险（trunk 分支不稳定将会阻碍在这个分支上工作的所有人）。

3. 执行实时 Code Review

当你研发的功能在本地验证通过之后，下一步就是寻找团队中其他人帮忙 Code Review。通常大家习惯发起一个 Pull Request，然后等待团队中的其他人进行 Code Review。由于各种原因，比如：没有及时看到这个 Pull Request，这个 Code Review 的过程将变得漫长起来。因此，在实践 Trunk-based Development 时，需要实时进行 Code Review，以便缩短 Code Review 所需的时间，进而能够及时将改动推送到 trunk 分支。这种实时 Code Review 的做法有很多种，比如：让你身后的团队成员到你的电脑前帮忙 Code Review，同时你可以给他解释为什么要这么做。

4. 使用 branch by abstraction 或者 feature flags 等技术来逐步提交还在研发中的功能

团队在研发的日常中，总是会遇到一些复杂且耗时的任务，完成这些任务需要一周或是更长的时间。因此在处理这些任务时不仅需要将其分解成多个子任务，而且每次完成子任务的研发时都需要将其提交到 trunk 分支中并且通过 branch by abstraction 或者 feature flags 等技术禁用这些子功能。直到所有子功能完成并且放在一起能够工作时才将这个任务开放出来。应用 branch by abstraction 或者 feature flags 等技术时应该遵守简单的原则，比如可以在代码中为该任务编写一个入口函数，但是这个入口函数没有被调用。

5. 每次向 trunk 提交代码时，都应该自动地触发编译和测试

为了能够自动化编译和测试新的提交，需要借助 CI 服务器。通过 CI 服务器构建编译和测试 2 个阶段（如下图所示）。这么做的目的是确保每一次提交都能够被机器自动化的验证，从而确保每一次提交都没有破坏 trunk。

![git_flow_6](..\asserts\git_flow_6.png)

6. 如果某一次提交破坏了 trunk 分支，那么应该停下手中的任务，优先恢复 trunk 分支

团队在日常的研发事务中总是会犯错，如果一次疏忽导致 trunk 分支无法通过测试，那么需要第一时间解决这个问题。如果这个问题无法快速解决，那么需要将此次提交撤销，并回退到上一次提交。这么做就是要确保 trunk 分支随时可用。

Trunk-Based Development 自身并没有给团队带来任何好处，为团队带来好处的是在 Trunk-Based Development 中应用了以上 6 点技巧。因此在企业研发部门实施 Trunk-Based Development 时，需要团队中的每一名成员都要掌握以上技巧，最终养成习惯。从这个角度来看，Trunk-Based Development更多的是依赖于人的行为，在一致的行为下应用自动化工具能够从整体上提高企业的研发效率！

当研发人员都掌握了以上技巧，同时，企业的研发部门已经决定使用 Trunk-Based Development 来研发产品，那么接下来就需要搭建一些自动化基础设施来辅助整个研发团队，其中 CI 服务就是构建现代化高效软件研发流程的初始环节。

## 为 Trunk-Based Development 配套 CI 服务

CI（Continues Integration）是指将各个研发团队的研发成果正确且快速地集成在一起，并提供给其他团队（测试团队、DevOps 团队等）使用。为了将 Trunk-Based Development 向整个研发部门推广，则需要一个好的 CI 服务。每一个提交到 trunk 上的改动，都会自动地触发 CI 服务，并由该服务获取 trunk 上的源码并顺序执行自定义的一些步骤。这些步骤有编译该源码和执行单元测试，每一步执行结束后都会输出一些结果，这些结果有成功或者失败，如果失败则会出现失败的信息。为了能够让团队成员及时看到失败的结果，一种做法是将在团队周围放置一台大电视，用于显示 CI 服务的执行结果或者发送失败的邮件给团队成员。

除了要搭建 CI 服务，还需要应用一些发布策略。比如：上图从 trunk 分支中拉出 release 分支，这么做是为了基于 release 分支对外发布产品，同时团队的其他成员依然能够在 trunk 上提交代码。由于 release 分支主要是为了对外发布产品，因此它不仅需要 CI 的支持，还需要 CD（Continuous Delivery）的支持，2 者结合就是 CI/CD。与 trunk 不同，CI 服务不仅需要监测 release 分支的变化并自动地编译源码、执行单元测试，而且还需要将编译的结果归档到团队内部共享的存储服务上，并自动地触发 CD 服务，使得 CD 服务能够将编译结果从存储服务中自动地部署到研发环境（dev）、测试环境（test）、预生产环境（stage）和生产环境（prod）。环境越多，实施自动化部署将任务也将增多，因此企业需要结合自身的现实状况来决定哪些环境是需要的。

## Trunk-Based Development 的实施细节

不同企业在研发团队中实施 Trunk-Based Development 都会有一些细微的差别，对于大多数需要研发软件产品的企业，可以参考以下步骤在研发团队中实施 Trunk-Based Development。

- 将产品相关的代码放到一个 repository 里，并且严格要求这个 repository 的分支数量每天不能超过研发人数；
- 该 repository 有一个长期存在的分支 trunk 或者 master。当需要对外发布的时候则需要拉出 release 分支，当有新的功能要发布的时候，将该分支删除并拉取新的 release 分支。研发人员可以直接在 trunk 或 master 分支上提交代码，当然也可以拉取 feature 分支，但是 feature 分支的生命周期应该在 1 天之内；
- 搭建 CI 服务，可以考虑使用 jenkins 或者使用 github actions。前者需要自己搭建，工作量大，服务器可以是自建，也可以使用云服务提供商的服务器。后者只需要编写 .yaml 文件就可以构建 CI 服务，服务器是 github 提供；
- CI 服务器会检测 trunk 和 release 分支，每个次提交，都会触发 CI 服务器，构建代码和执行自动化测试。trunk 分支所对应的 CI 构建流程，其运行一次所需要的时间需要控制在 30 分钟之内，其目的是为了检测每次提交都是正常的。release 分支所对应的 CI 构建流程，其运行一次所需要的时间也需要控制在 30 分钟之内并归档编译出来的结果，除此之外还需为该分支搭建 CD 服务。CD 服务能够将编译出来的结果自动部署到其它环境；
- 为团队的每一个研发人员预留时间学习和掌握之前提到的技巧，使得团队成员达成共识；
- 每次发布只能通过 release 分支，修复 bug 和性能优化的改动应该提交到 trunk 分支，最终通过 cherry-pick 的方式将这些提交merge 到 release 分支。当有新功能对外发布时，需要删除原来的 release 分支，并从 trunk 分支拉取新的 release 分支。

## 使用 github 来实施 Trunk-Based Development 的基本思路

github 提供了大多数功能给开发者使用，这些功能有：账号管理、源码托管、Action 服务（也就是 CI/CD）、文件存储和协作沟通的线上通道。

在 github 上实践 Trunk-Based Development 的基本思路：

1. 在 github 上创建一个 repository，这个 repository 用于放置产品的源码；
2. 为 master 和 release 分支创建对应的 CI 服务，也就是在 .github/workflows 目录中创建 2 个文件：master.yml 和 release.yml。每个文件定义了一个 workflow，每个 workflow 定义了触发条件和一些执行步骤。master.yml 针对 master 分支定义了 build，test 步骤，每次提交到该分支都会执行  build 和 test 步骤；release.yml 针对 release 分支定义了 build，test，archive，每次在 release 分支上打 tag（比如：v0.0.1）都会在该分支上执行 build，test 和 archive步骤。将创建的 master.yml 和 release.yml 文件推送到该repository 上；
3. 将每一个参与研发的人员加入到这个 repository 中，并授予他们可读写的权限；
4. 研发人员可以直接在 master 分支上提交，也可以拉出 feature 分支进行修改，最终合并到 master 分支，但是这个 feature 分支的生命周期不能超过1天。当研发人员在 master 分支上提交代码后，会自动触发 master.yml 所定义的 workflow。该 workflow 将执行 build 和 test 步骤来确保 master 分支是正常的；
5. 如果使用 feature 分支，则会用到 github 的 pull request 功能。这个功能可以帮助团队成员进行 Code Review。当某一名成员发起pull request 时，同样也会触发 master.yml 所定义的 workflow。团队的其他成员则可以在 pull request 的操作面板上提交意见，查看此次发起提交的运行结果；
6. 当需要对外发布的时候，那么可以拉取另外一个分支 release ，然后对该分支打上 tag（比如：v0.0.1），此时 release.yml 所定义的workflow 会启动，该 workflow 除了执行步骤 build 和 test，最终还要执行 archive 步骤。执行 archive 的时候会把生成的结果发布到 github 的 release 存储。release 存储是对外开放的，任何人都可以到 release 存储获取并使用你发布的产品；
7. 当发布的产品有 bug 时（比如功能缺陷，性能差劲等），可以通过 cherry-pick 从 master 分支选取对应的 commit 合并到 release分支。当已知的 bug 都修复了，那么在 release 分支上打上一个 tag（比如：v0.0.2），此时会触发 release.yml 所定义的 workflow；
8. 当进入下一次迭代并准备好发布新功能的时候，那么需要把之前的 release 分支删除，并且再从分支 master 拉出新的 release 分支，此时产品的版本号应该变成 v0.1.x。

通过以上思路便可以在 github 上创建一个高效的基于 Trunk-Based Development 的 CI 流程，如下图所示：

![git_flow_7](..\assets\git_flow_7.png)

# 结论

Trunk-based Development 已经被各大公司（Google、Facebook、LinkedIn 等）成功实践了十几年。企业在研发产品时，想要在研发部门中顺利地实施 Trunk-based Development，还需要掌握一些技巧和搭建自动化基础设施。这些技巧需要所有研发人员达成共识，并养成习惯。

当习惯形成之后，则需要借助一些自动化基础设施来加速研发流程，这个研发流程就是 CI/CD。目前有许多成熟的工具提供 CI 的支持，比如：使用 github 提供的服务来快速搭建 CI 流程（具体参考链接 [1]）。

当研发流程搭建起来之后，则需要应用一些发布策略。一切就绪之后，整个研发团队的研发效率将会达到一个质的飞跃。在研发团队中实施 Trunk-based Development 以及为其搭建 CI 服务只是构建企业级软件研发流程的第一步。当新功能完成研发，并准备好发布的时候，也需要一套基础设施将研发好的功能及时高效地发布到用户现场，这就是 CD（具体参考链接 [2]）。

# 参考

[1]: https://2cloudlab.com/blog/how-to-setup-ci-service-base-on-github/	"如何 0 成本在 gitthub 上 CI"
[2]: https://2cloudlab.com/blog/why-organization-should-practice-cicd/	"如何提高企业的研发效率--CI/CD"