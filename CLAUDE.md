# 项目说明

## 可用工具：图像生成 API（只用于材质层）

本环境通过环境变量提供了一个兼容 OpenAI Images API 的第三方中转站，可以生成位图。它在本项目里唯一的用途是生成"材质层"：纸面纹理、纤维网络的抽象噪声、显微光场的明暗、可平铺的底纹贴图。不要用它生成插图、假显微照片、植物、人物、发光科幻图或带文字的图。界面去掉这些贴图后必须仍然成立，所以每张图都要有程序化（Canvas / GLSL 噪声）替代方案。

相关环境变量是 `IMAGE_API_BASE_URL`（`https://yostoken.top/v1`）、`IMAGE_API_KEY` 和 `IMAGE_MODEL`（默认 `gpt-image-2`）。如果 `IMAGE_API_KEY` 未设置，告诉用户需要在云环境设置里添加，然后直接用程序化材质继续工作，不要让用户把 key 贴进聊天。

### 调用方式

```bash
curl -sS --max-time 180 "$IMAGE_API_BASE_URL/images/generations" \
  -H "Authorization: Bearer $IMAGE_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"model\":\"$IMAGE_MODEL\",\"prompt\":\"seamless tileable grayscale paper fiber texture, low contrast, no text\",\"size\":\"1024x1024\",\"n\":1}" \
  -o "<scratchpad 目录>/tex_raw.json"
```

响应里的图片可能在 `data[0].b64_json`（base64，解码即可），也可能在 `data[0].url`（再用 curl 下载）。两种都要处理。`/images/edits` 路由也存在，可用于在已有贴图上做变体。

### 已验证与未验证

已验证：中转站可达；key 有效；`GET /models` 列出 19 个模型，全是图像模型（`gpt-image-2`、`gpt-image-2.5`、`seedream-5.0-pro`、`gemini-3-pro-image`、`nano-banana-pro` 等），没有文本模型；`/images/generations` 收到缺 prompt 的请求时返回 422 `openai_error`，这是上游的校验错误，不代表路由不存在；不需要任何额外请求头。

未验证：实际生成一张图能否成功。第一次调用请先用一张小图测试；如果 `IMAGE_MODEL` 报错，依次换 `gpt-image-2.5`、`seedream-5.0-pro` 试，并把可用的模型告诉用户，让用户更新环境变量。`size`、`quality` 等参数按 OpenAI Images API 的写法传；如果中转站拒绝某个参数，就去掉它。

### 预算与安全

这把 key 的额度有限。每张图都要花钱，所以动手前先规划好要哪几张，整个任务最多生成 3 到 5 张，不要循环重试，失败两次就改用程序化材质。生成结果先存到 scratchpad，后续改版复用已有的图，不要重新生成。

永远不要回显、打印或把 key 写进任何文件，也不要提交它，只通过 `$IMAGE_API_KEY` 引用。中转站是第三方服务，运营方能看到请求内容，prompt 里只写材质描述。

### 后处理

生成模型不保证无缝平铺，拿到图后用 PIL 和 numpy（环境已预装）处理：转灰度；把图平移半个宽高（`np.roll`）得到边缘连续的版本，再用中心为 1、边缘为 0 的平滑窗口与原图混合，消除接缝；压低对比度；缩到 512px 左右，编码成 WebP 或 JPEG，以 base64 data URI 内嵌进单文件 HTML。
