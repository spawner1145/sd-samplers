# sd-samplers
一些乱写的smea(?)采样器，我是业余的菜鸡一只，如果有大佬有更好的思路请给我pr(

# 如果你是forge或者comfyui这个仓库直接可以当成插件和自定义节点用

# 不过我发现直接当webui/forge插件加和代码里面你手动加效果不一样，原因未知，我个人感觉还是直接手动加好一点，看original.py里面注释的提示，把代码加到相应的位置，具体我放在后面，comfyui无此问题，放心把这个仓库放入custom_nodes文件夹食用

<p align="center">
    <img src="https://github.com/user-attachments/assets/d9490d19-f423-4bd6-92c8-95a41c843707" width="150" style="margin: 5px;">
    <img src="https://github.com/user-attachments/assets/24a45098-0fa2-452c-9a32-c70760fd24d2" width="150" style="margin: 5px;">
    <img src="https://github.com/user-attachments/assets/e4be836e-1cae-4381-8a37-0ce62591e252" width="150" style="margin: 5px;">
    <img src="https://github.com/user-attachments/assets/582d26fe-3251-4a94-bb9b-efb604b692af" width="150" style="margin: 5px;">
    <img src="https://github.com/user-attachments/assets/a67788ff-f790-4337-9472-de9993152686" width="150" style="margin: 5px;">
</p>

先后依次是`euler，euler a，Spawner SMEA，Spawner SMEA (beta)，Spawner SMEA Dyn (beta)`，注意`Spawner SMEA，Spawner SMEA (beta)，Spawner SMEA Dyn (beta)`这三个我自己瞎jb写的，不代表novel ai官方逻辑，用的模型是noob eps 1.1，这里的测试样例是手动加代码而不是插件的，如果你想要复现需要手动加代码(见original.py)，调度器如果想要轮廓分明推荐simple，如果想要羽化边缘的感觉用karras，不过你webui用automatic也行(

在上述采样器外，还有`Spawner SMEA Dyn (beta1)`，相比于`Spawner SMEA Dyn (beta)`，对一些参数进行了微调，具体来说：eta_start 0.95->0.98，eta_end 0.70->0.65。这会在采样初期增加更多随机性（探索更多可能性），而在后期减少随机性（更精确地细化细节），一般来说start越高，end越低，产生的多样性就越强。调整 eta_exponent 为1.0->1.5，这会使从高噪声到低噪声的过渡更加陡峭，可能会产生更清晰的细节。在问卷调查中，普遍出现了`Spawner SMEA Dyn (beta1)`优于`Spawner SMEA Dyn (beta)`的反馈，未来你可以自行修改采样器参数，具体来说：
```
eta_start 和 eta_end：这两个参数控制采样过程中噪声添加的量
eta_start 控制采样初期的噪声量
eta_end 控制采样后期的噪声量
eta_exponent：控制从 eta_start 到 eta_end 的过渡曲线
增大这个值会使过渡更加陡峭
beta：控制连续去噪估计的混合比例
这个参数影响当前步骤和前一步骤的去噪估计的混合
s_noise：控制添加噪声的整体缩放
```

具体参数
```
shiroko (blue archive),multiple girls,2girls,animal ears,shiroko terror (blue archive),wolf ears,v,breasts,halo,cross hair ornament,grey hair,mismatched pupils,animal ear fluff,gloves,blue eyes,extra ears,cleavage,double v,black choker,hair ornament,long hair,choker,scarf,white background,jacket,long sleeves,simple background,black gloves,looking at viewer,large breasts,black dress,dress,green gloves,hair between eyes,blue scarf,closed mouth,hand on another's chin,school uniform,blazer,diamond-shaped pupils,ahoge,upper body,medium hair,broken halo,cheek squash,collarbone,black jacket,pout,diamond (shape),blush,buttoned cuffs,dual persona,shirt,
(artist:reoen:0.826446),artist:machi,(artist:ningen_mame:0.9),(artist:sho_(sho_lwlw):0.9),(artist:rhasta:0.9),(artist:wlop:0.7),(artist:ke-ta:0.6),(fkey:0.5),(tianliang duohe fangdongye:0.5),(hiten \(hitenkei\):0.6),best quality,amazing quality,(artist:onineko:0.826446),very aesthetic,
masterpiece,year 2024,newest,highres,absurdres,
Negative prompt: worst aesthetic,worst quality,old,early,low quality,bad quality,lowres,signature,username,bad id,bad twitter id,english commentary,logo,bad hands,mutated hands,mammal,anthro,furry,ambiguous_form,feral,semi-anthro,
Steps: 28, Sampler: Spawner SMEA Dyn (beta), Schedule type: Automatic, CFG scale: 4.5, Seed: 114514, Size: 1024x1536, Model: noobaiXLNAIXL_epsilonPred11Version, Clip skip: 2, RNG: CPU, Version: f2.0.1v1.10.1-1.10.1
```

## 【2025.5.26】更新了`spawner_rk2_smea_d_clamp`，主要适用于dit架构模型，解决了先前采样器过曝的问题，10步就可以获得高质量图像

### 主要特点

模糊矫正机制：通过在每次迭代中应用高斯模糊来平滑输出，防止生成过程中出现突变或不自然的变化。模糊强度会根据当前噪声水平动态调整，并且设定了一个min_blur_sigma值，确保即使在采样的最后阶段也不会完全取消模糊矫正（可以有效解决过拟合模型的水晶状噪声问题）

噪声调度：允许对添加到中间结果中的噪声量进行控制。通过参数eta控制噪声的比例，并使用noise_sampler函数来生成噪声

历史信息利用：引入了一个beta参数，用来决定是否以及如何将前一步的结果与当前步的结果结合，以产生更稳定的更新方向。这有助于平稳过渡并提高生成质量

自适应步长：虽然基于RK2(Runge-Kutta 2 阶)，但该方法还考虑了从一种噪声水平到另一种噪声水平的“祖先步骤”(get_ancestral_step)，使得每一步的大小不是固定的，而是根据当前状态动态调整

### 工作机制

初始化：设置初始条件，包括确定最大噪声水平max_sigma、创建单位张量s_in等

主循环：遍历所有预定的噪声水平（sigmas），对于每个噪声水平:
- 首先计算两个中间点（k1, k2），其中k1基于当前噪声水平下的模型预测结果，而k2则是在一个中间状态下再次评估得到
- 在这两个点之间，应用模糊处理并计算出相应的梯度或更新方向
- 使用这些信息更新样本x，同时根据需要添加随机噪声
- 结束条件：当遍历完所有预设的噪声水平后，返回最终生成的样本

这种设计旨在平衡效率与生成质量，逐步细化样本，直到达到所需的清晰度和细节水平。此外，还特别注意避免生成过程中的不稳定现象，如末端突变，通过持续的模糊矫正来保证输出的平滑性和一致性

# 手动改代码方法
`step 1`: 打开repositories\k-diffusion\k_diffusion\sampling.py(<-这是普通webui，forge的话是k_diffusion\sampling.py，comfyui是comfy\k_diffusion\sampling.py)

<p align="center">
    <img src="https://github.com/user-attachments/assets/67abb38b-dfc0-463b-95b9-9e4585c59f27" width="1000" style="margin: 5px;">
</p>

`step 2`: 复制本仓库original.py中这条长横线前面的所有代码，并粘贴到sampling.py的末尾

<p align="center">
    <img src="https://github.com/user-attachments/assets/cc87f890-f422-4330-8966-a493255a2e48" width="1000" style="margin: 5px;">
    <img src="https://github.com/user-attachments/assets/46f2890d-b92d-44d0-a5ed-7b600a37b588" width="1000" style="margin: 5px;">
</p>

## 如果你是comfyui:

`Step 3`: 打开comfy\samplers.py，并找到KSAMPLER_NAMES，大概在709行附近

<p align="center">
    <img src="https://github.com/user-attachments/assets/6c4ba935-2990-4a19-8ad9-337facd15e3b" width="1000" style="margin: 5px;">
</p>

`Step 4`: 在列表中添加"spawner_smea"，"spawner_smea_beta"，"spawner_smea_dyn_beta"，就像这样:
```
KSAMPLER_NAMES = ["euler", "euler_cfg_pp", "euler_ancestral", "euler_ancestral_cfg_pp", "heun", "heunpp2","dpm_2", "dpm_2_ancestral",
                  "lms", "dpm_fast", "dpm_adaptive", "dpmpp_2s_ancestral", "dpmpp_2s_ancestral_cfg_pp", "dpmpp_sde", "dpmpp_sde_gpu",
                  "dpmpp_2m", "dpmpp_2m_cfg_pp", "dpmpp_2m_sde", "dpmpp_2m_sde_gpu", "dpmpp_3m_sde", "dpmpp_3m_sde_gpu", "ddpm", "lcm",
                  "ipndm", "ipndm_v", "deis", "res_multistep", "res_multistep_cfg_pp", "res_multistep_ancestral", "res_multistep_ancestral_cfg_pp",
                  "gradient_estimation", "gradient_estimation_cfg_pp", "er_sde", "seeds_2", "seeds_3", "spawner_smea", "spawner_smea_beta", "spawner_smea_dyn_beta"]
```
然后就能用了

## 如果你是webui或者forge:

`Step 3`: 打开modules\sd_samplers_kdiffusion.py，并找到samplers_k_diffusion和sampler_extra_params这两个地方

<p align="center">
    <img src="https://github.com/user-attachments/assets/5be7e824-a7f6-46f4-959e-13e12fba5e52" width="1000" style="margin: 5px;">
    <img src="https://github.com/user-attachments/assets/9b9045db-3bea-4e42-815b-c0ab59356692" width="1000" style="margin: 5px;">
</p>

`Step 4`: original.py中找到对应部分，并替换原来modules\sd_samplers_kdiffusion.py的

<p align="center">
    <img src="https://github.com/user-attachments/assets/134664e0-d48a-4c2b-83c4-179d6447386d" width="1000" style="margin: 5px;">
    <img src="https://github.com/user-attachments/assets/502d2e09-c66f-40e6-9514-73cf70ad3d29" width="1000" style="margin: 5px;">
</p>

`Step 5`: 启动webui或者forge，你将看到如图，带spawner开头的就是

<p align="center">
    <img src="https://github.com/user-attachments/assets/a3e2d798-32b4-454d-aff0-d679dc6d45ea" width="1000" style="margin: 5px;">
</p>

## 用grok水了一个概览(因为懒得写具体的.jpg,要看原理的自己看吧(
你也可以看这里 https://deepwiki.com/spawner1145/sd-samplers 这个网站基本讲解清楚了代码的思路
---

## Enhanced Ancestral Sampling Methods for Diffusion Models via Consecutive Denoised Estimate Averaging and Dynamic Noise Control

**Abstract**

Denoising Diffusion Probabilistic Models (DDPMs) and their variants have shown remarkable performance in generative tasks. A crucial aspect of these models is the sampling process, which typically involves iteratively denoising a noisy sample according to a predefined noise schedule. Ancestral sampling, a common technique, involves adding noise at each step to maintain stochasticity. This paper presents and analyzes three related ancestral sampling methods, implemented in Python, which introduce specific modifications to the standard process: 1) Simple averaging of consecutive denoised estimates (SMEA), 2) A parameterized version of this averaging (SMEA-beta), and 3) A method combining SMEA-beta with a dynamic eta parameter that controls the amount of added noise based on the current noise level (SMEA-Dyn-beta). We detail the mechanics of each method, highlighting the mechanisms for smoothing the denoised predictions and dynamically controlling the stochasticity of the sampling trajectory.

**1. Introduction**

Diffusion models have emerged as powerful generative models, achieving state-of-the-art results in image, audio, and video synthesis. They operate by learning to reverse a diffusion process that gradually adds noise to data. The core challenge after training is the efficient and high-quality generation of new samples, which requires navigating the learned reverse process.

Sampling methods for diffusion models can broadly be categorized into methods based on ordinary differential equations (ODEs) and stochastic differential equations (SDEs) that the diffusion process can be mapped to [1, 2]. Ancestral sampling is a technique typically associated with SDE-based samplers or discrete-time approximations that involve adding noise at each step, offering a balance between sample quality and diversity compared to deterministic ODE solvers like DDIM [3]. The amount of noise added in ancestral sampling is often controlled by a parameter, commonly denoted as $\eta$ (eta), where $\eta=0$ corresponds to a deterministic DDIM-like step and $\eta=1$ corresponds to a DDPM-like step (when using appropriate noise schedules).

This paper examines three closely related ancestral sampling implementations: `sample_spawner_smea`, `sample_spawner_smea_beta`, and `sample_spawner_smea_dyn_beta`. These methods introduce novel modifications focusing on how the model's prediction of the clean data ($x_0$) is utilized and how the $\eta$ parameter is determined across the sampling steps.

**2. Background: Ancestral Sampling**

Ancestral sampling proceeds by iteratively denoising a sample $x_t$ corrupted with noise level $\sigma_t$ to obtain a sample $x_{t-\Delta t}$ with a lower noise level $\sigma_{t-\Delta t}$. A standard step involves predicting the clean data $\hat{x}_0$ from $x_t$ using the trained model. This prediction is then used to estimate the score or the direction of the reverse step. Unlike deterministic solvers which move directly towards the predicted $x_0$, ancestral sampling adds a controlled amount of noise at each step.

Given a noisy sample $x_i$ at noise level $\sigma_i$ and the model's prediction of the clean data $\hat{x}_0^{(i)}$, the direction $d_i$ can be computed as $d_i = (x_i - \hat{x}_0^{(i)}) / \sigma_i$. The update step often involves moving towards the predicted clean data direction, followed by adding noise:

$x_{i+1} = x_i + d_i \cdot (\sigma_{\text{down}} - \sigma_i) + \text{noise} \cdot \sigma_{\text{up}}$

where $\sigma_{\text{down}}$ and $\sigma_{\text{up}}$ are components derived from $\sigma_i$, $\sigma_{i+1}$ (the next noise level), and the $\eta$ parameter, typically satisfying $\sigma_{\text{down}}^2 + \sigma_{\text{up}}^2 = \sigma_i^2 - \sigma_{i+1}^2$. The function `get_ancestral_step(sigma_i, sigma_{i+1}, eta)` likely calculates these $\sigma_{\text{down}}$ and $\sigma_{\text{up}}$ values.

The methods presented here build upon this ancestral sampling framework by modifying how $\hat{x}_0^{(i)}$ is used in calculating $d_i$ and how $\eta$ is determined.

**3. Proposed Methods**

The three proposed sampling methods are variations of ancestral sampling that primarily differ in how the denoised estimate is processed and how the $\eta$ parameter is set.

**3.1. SMEA with Alpha Blending (`sample_spawner_smea`)**

This method introduces a form of averaging the denoised estimates across consecutive steps. Instead of directly using the current step's denoised prediction $\hat{x}_0^{(i)}$ to compute the step direction, it computes an *effective* denoised estimate $\hat{x}_0^{\text{eff}(i)}$ by blending the current prediction with the prediction from the previous step, $\hat{x}_0^{(i-1)}$.

The core update to the denoised estimate is:

$$
\hat{x}_0^{\text{eff}(i)} = (1 - \alpha) \hat{x}_0^{(i)} + \alpha \hat{x}_0^{(i-1)}
$$

where $\hat{x}_0^{(i)}$ is the output of the model at step $i$, and $\hat{x}_0^{(i-1)}$ is the model output from step $i-1$. The parameter $\alpha \in [0, 1]$ controls the weight given to the previous estimate. If $\alpha = 0$, this reduces to using only the current prediction, like standard ancestral sampling. If $\alpha > 0$, it incorporates information from the previous step's prediction. This averaging is applied only if `old_denoised` (representing $\hat{x}_0^{(i-1)}$) is available and the next noise level $\sigma_{i+1}$ is greater than 0 (i.e., not the final step).

The step direction is then computed using this effective estimate:

$$
d_i = (x_i - \hat{x}_0^{\text{eff}(i)}) / \sigma_i
$$

The sampling proceeds with the standard ancestral update using `get_ancestral_step` with a fixed $\eta$ parameter and adding noise scaled by $\sigma_{\text{up}}$.

The motivation behind averaging consecutive denoised estimates could be to smooth the sampling trajectory by reducing step-to-step variance in the model's prediction, potentially leading to more stable or higher-quality samples.

**3.2. SMEA with Beta Parameterization (`sample_spawner_smea_beta`)**

This method is structurally very similar to the alpha-blending SMEA but uses a parameter named `beta` instead of `alpha` for the blending weight, with a default value of 0.55 (compared to alpha's default of 0.5, although not explicitly shown in the first function signature, it's a common default). The blending formula is identical:

$$
\hat{x}_0^{\text{eff}(i)} = (1 - \beta) \hat{x}_0^{(i)} + \beta \hat{x}_0^{(i-1)}
$$

where $\beta \in [0, 1]$. The blending is also conditioned on `old_denoised_for_beta` being available and $\sigma_{i+1} < \sigma_i$ (moving to a lower noise level).

This version fixes the $\eta$ parameter for the ancestral step to a default value of 0.85. The rest of the sampling process, including the calculation of $d_i$ using $\hat{x}_0^{\text{eff}(i)}$ and the ancestral update with noise addition, follows the standard approach based on the fixed $\eta$.

This method appears to be a specific configuration or a slight variation of the alpha-blending SMEA, possibly tuned with specific default parameters ($\beta=0.55$, $\eta=0.85$).

**3.3. SMEA with Dynamic Eta (`sample_spawner_smea_dyn_beta`)**

The third method combines the beta-parameterized SMEA denoised estimate blending with a *dynamic* $\eta$ parameter. The blending of consecutive denoised estimates using the $\beta$ parameter is identical to the `sample_spawner_smea_beta` method:

$$
\hat{x}_0^{\text{eff}(i)} = (1 - \beta) \hat{x}_0^{(i)} + \beta \hat{x}_0^{(i-1)}
$$

However, instead of using a fixed $\eta$ for the ancestral step, this method calculates a `current_eta` value for each step $i$ based on the current noise level $\sigma_i$. The $\eta$ value interpolates between a starting value `eta_start` and an ending value `eta_end` based on the relative position of $\sigma_i$ in the noise schedule, potentially weighted by an exponent `eta_exponent`:

$$
\text{current\_sigma\_ratio} = \left(\frac{\sigma_i}{\sigma_{\text{max}}}\right)^{\text{eta\_exponent}}
$$

$$
\text{current\_eta} = \text{eta\_end} + (\text{eta\_start} - \text{eta\_end}) \cdot \text{current\_sigma\_ratio}
$$

where $\sigma_{\text{max}}$ is the maximum noise level in the schedule (or a specified `sigma_max_for_dyn_eta`). The `current_sigma_ratio` is clamped between 0 and 1. This means $\eta$ will be closer to `eta_start` at high noise levels and closer to `eta_end` at low noise levels. The `eta_exponent` allows for controlling the curve of this interpolation.

The `get_ancestral_step` function then uses this `current_eta` to determine $\sigma_{\text{down}}$ and $\sigma_{\text{up}}$, thus dynamically controlling the amount of noise added at each step. For example, setting `eta_start` higher than `eta_end` would lead to more stochasticity (more added noise) at the beginning of the sampling process (high $\sigma$) and more determinism towards the end (low $\sigma$).

This dynamic control over $\eta$ allows for fine-tuning the balance between sample diversity (higher $\eta$) and sample fidelity (lower $\eta$) throughout the denoising process, potentially exploiting the different characteristics of the model's predictions at various noise levels.

**4. Comparison and Discussion**

The three methods represent an evolution of ancestral sampling with specific enhancements:

| Feature                    | `sample_spawner_smea`                                  | `sample_spawner_smea_beta`                           | `sample_spawner_smea_dyn_beta`                       |
| :------------------------- | :------------------------------------------------------- | :----------------------------------------------------- | :----------------------------------------------------- |
| Denoised Estimate Blending | Alpha-weighted average                                   | Beta-weighted average                                  | Beta-weighted average                                  |
| Blending Parameter Name    | `alpha`                                                | `beta`                                               | `beta`                                               |
| Blending Formula           | $(1-\alpha)\hat{x}_0^{(i)} + \alpha \hat{x}_0^{(i-1)}$ | $(1-\beta)\hat{x}_0^{(i)} + \beta \hat{x}_0^{(i-1)}$ | $(1-\beta)\hat{x}_0^{(i)} + \beta \hat{x}_0^{(i-1)}$ |
| Eta Parameter              | Fixed (`eta` parameter)                                | Fixed (default `eta=0.85`)                           | Dynamic (`eta_start`, `eta_end`, `eta_exponent`) |
| Dynamic Eta based on Sigma | No                                                       | No                                                     | Yes                                                    |

The core innovation across these methods appears to be the "SMEA" concept – averaging the current and previous step's $x_0$ predictions. This can be seen as a form of momentum or smoothing applied to the predicted signal, potentially improving stability, especially if the model's predictions are noisy or inconsistent between adjacent steps.

The introduction of `beta` in the second and third methods seems to be a re-parameterization or refinement of the alpha blending, perhaps with preferred default values derived from empirical tuning.

The most significant addition in the third method is the dynamic $\eta$. This allows for a flexible sampling strategy where the level of stochasticity can be adapted to the noise level. For instance, a higher $\eta$ at high noise might help explore diverse modes, while a lower $\eta$ at low noise might refine details and improve fidelity. The `eta_exponent` provides further control over the profile of this dynamic adaptation.

**5. Conclusion**

We have described three ancestral sampling methods for diffusion models: SMEA, SMEA-beta, and SMEA-Dyn-beta. These methods extend standard ancestral sampling by incorporating an averaging mechanism for consecutive denoised estimates and, in the case of SMEA-Dyn-beta, a dynamic strategy for setting the $\eta$ parameter based on the current noise level. The blending of denoised estimates aims to provide a more stable or consistent signal for the reverse process, while the dynamic $\eta$ allows for tailoring the balance between stochasticity and determinism throughout the sampling trajectory.

Future work should involve thorough experimental evaluation of these methods on various diffusion models and datasets to quantify their impact on sample quality, diversity, and computational efficiency compared to existing state-of-the-art samplers. Theoretical analysis of the averaging and dynamic eta mechanisms could also provide deeper insights into their effects on the sampling path and the properties of the generated samples.

**References**

[1] Song, Y., Meng, C., & Ermon, S. (2020). Denoising Diffusion Implicit Models. *arXiv preprint arXiv:2010.02502*.
[2] Song, Y., Kingma, D. P., Ermon, S., Kumar, S., & Poole, B. (2021). Score-Based Generative Modeling through Stochastic Differential Equations. *arXiv preprint arXiv:2011.13456*.
[3] Nichol, A. Q., & Dhariwal, P. (2021). Improved Denoising Diffusion Probabilistic Models. *arXiv preprint arXiv:2102.09672*.
*(Note: These are placeholder references for common diffusion sampling concepts. Specific references for the SMEA technique or dynamic eta as implemented would be needed if they exist in prior work)*

**Code Availability**

The methods described in this paper are based on the provided Python code snippets. The full implementation, including the `get_ancestral_step`, `to_d`, and `default_noise_sampler` functions, would be required for reproduction and further study.

author@spawner1145
---
