---
title: "Whisper AI CoreML 패키지 개발 실패 경험담"
description: "Whisper AI CoreML 패키지 개발 실패 경험담"
categories: App
tags: [개발, 앱, spm, 모듈]
sidebar: 
  nav: "docs"
header:
  teaser: https://www.google.com/url?sa=i&url=https%3A%2F%2Fzilliz.com%2Fglossary%2Fopenai-whisper&psig=AOvVaw2lO5E2-WGBzlQv6q57aQBf&ust=1742460146526000&source=images&cd=vfe&opi=89978449&ved=0CBQQjRxqFwoTCPi08oXglYwDFQAAAAAdAAAAABAE
---
# 눈물의 WhisperCoreML 패키지 제작 실패기
## 개발 동기
OpenAI 에서 제공하는 오픈소스인 Whisper AI를 이용해서 음성파일을 통해 자막 포맷의 파일을 만들어내는 중에 CLI가 아닌 GUI 형태의 앱이 있었으면 좋겠다는 생각을 했다. 나와 같은 생각을 한 사람이 나뿐은 아니었고 이미 수많은 사람들이 같은 생각으로 만든 앱이 있었다.([WhisperAI 활용 포럼](https://github.com/openai/whisper/discussions/categories/show-and-tell)) 하지만 괜한오기? + 나도 만들 수 있을 것같은데 내가 원하는 기능들을 넣어서 그리고 SwiftUI로 만들어서 앱스토어에 등록하고 또 파이널 컷 프로에 조금더 최적화 시킬 수 있지 않을까하는 호기심에서 시작했다. 하지만 whisper AI는 pytorch 형식으로 되어있고 이것을 구동하는 코드 또한 파이썬 코드였기에 coreml 형식으로 바꿔서 Swift 코드로 인코딩 및 디코딩 로직을 구현해내고 또 지원하는 기능들 또한 구현해야했다. 그래서 평소 모듈화에 관심이 있던 차에 내가 한번 해자는 생각으로 시작하게 되었다.

## 과정

우선 모델 파일을 구해야했는데 개인적으로 [huggingface](https://huggingface.co/openai)라는 것 조차 처음 알게 됐지만 수많은 AI 모델들이 올라와 있는 hub 가 있었고 거기서 모델을 다운받아서 변환시켰다.

다음과 같은 파이썬 코드로 변환하면 mlpackage 형식으로 출력이 된다.
```python

import os
import torch
import numpy as np
import coremltools as ct
from transformers import WhisperForConditionalGeneration, WhisperConfig
from safetensors.torch import load_file

def convert_tiny_safetensors_to_mlpackage(safetensors_filename: str,
                                          config_filename: str,
                                          output_encoder_path: str,
                                          output_decoder_path: str):
    # 현재 스크립트가 위치한 디렉토리를 기준으로 경로 설정
    current_dir = os.path.dirname(os.path.abspath(__file__))
    safetensors_path = os.path.join(current_dir, safetensors_filename)
    config_path = os.path.join(current_dir, config_filename)

    print("Tiny 모델 구성 로드 중...")
    config = WhisperConfig.from_json_file(config_path)
    model = WhisperForConditionalGeneration(config)

    print("Tiny 모델 가중치 로드 중...")
    state_dict = load_file(safetensors_path)
    # 누락된 키가 있을 경우 strict=False로 로드
    model.load_state_dict(state_dict, strict=False)
    model.eval()

    print("인코더/디코더 분리 중...")
    encoder = model.get_encoder()
    decoder = model.get_decoder()
    lm_head = model.proj_out

    # 인코더 래퍼: encoder의 last_hidden_state만 반환
    class EncoderWrapper(torch.nn.Module):
        def __init__(self, encoder):
            super().__init__()
            self.encoder = encoder

        def forward(self, input_features):
            outputs = self.encoder(input_features)
            return outputs.last_hidden_state

    # 디코더 래퍼: decoder와 LM 헤드를 함께 실행하여 logits 반환
    class DecoderWrapper(torch.nn.Module):
        def __init__(self, decoder, lm_head):
            super().__init__()
            self.decoder = decoder
            self.lm_head = lm_head

        def forward(self, input_ids, encoder_hidden_states):
            outputs = self.decoder(
                input_ids=input_ids,
                encoder_hidden_states=encoder_hidden_states,
                use_cache=False,
                return_dict=True
            )
            logits = self.lm_head(outputs.last_hidden_state)
            return logits

    # 더미 입력 생성 (Tiny 모델: num_mel_bins=80, 입력 길이=3000)
    dummy_input = torch.rand(1, config.num_mel_bins, 3000, dtype=torch.float32)
    encoder_wrapper = EncoderWrapper(encoder)
    encoder_wrapper.eval()
    print("인코더 TorchScript 변환 중...")
    traced_encoder = torch.jit.trace(encoder_wrapper, dummy_input, strict=False, check_trace=False)

    print("인코더 CoreML 변환 중...")
    coreml_encoder = ct.convert(
        traced_encoder,
        inputs=[ct.TensorType(name="input_features", shape=dummy_input.shape, dtype=np.float32)],
        convert_to="mlprogram",
        compute_units=ct.ComputeUnit.ALL,
        minimum_deployment_target=ct.target.macOS13
    )

    print("인코더 모델 컴파일 중...")
    # compile_model()는 이미 컴파일된 모델이면 에러를 발생하므로 try/except로 처리합니다.
    try:
        compiled_encoder_url = ct.utils.compile_model(coreml_encoder)
        os.replace(str(compiled_encoder_url), os.path.join(current_dir, output_encoder_path))
    except TypeError:
        # 이미 컴파일되어 있다면 output_encoder_path로 직접 저장
        coreml_encoder.save(os.path.join(current_dir, output_encoder_path))
    print("인코더 모델 저장 완료:", os.path.join(current_dir, output_encoder_path))

    with torch.no_grad():
        encoder_output = encoder_wrapper(dummy_input)
    encoder_shape = encoder_output.shape

    dummy_input_ids = torch.ones((1, 1), dtype=torch.int32)
    decoder_wrapper = DecoderWrapper(decoder, lm_head)
    decoder_wrapper.eval()
    print("디코더 TorchScript 변환 중...")
    traced_decoder = torch.jit.trace(decoder_wrapper, (dummy_input_ids, encoder_output), strict=False, check_trace=False)

    print("디코더 CoreML 변환 중...")
    coreml_decoder = ct.convert(
        traced_decoder,
        inputs=[
            ct.TensorType(name="input_ids", shape=dummy_input_ids.shape, dtype=np.int32),
            ct.TensorType(name="encoder_hidden_states", shape=encoder_shape, dtype=np.float32)
        ],
        convert_to="mlprogram",
        compute_units=ct.ComputeUnit.ALL,
        minimum_deployment_target=ct.target.macOS13
    )
    print("디코더 모델 컴파일 중...")
    try:
        compiled_decoder_url = ct.utils.compile_model(coreml_decoder)
        os.replace(str(compiled_decoder_url), os.path.join(current_dir, output_decoder_path))
    except TypeError:
        coreml_decoder.save(os.path.join(current_dir, output_decoder_path))
    print("디코더 모델 저장 완료:", os.path.join(current_dir, output_decoder_path))

    print("변환 완료!")

if __name__ == "__main__":
    convert_tiny_safetensors_to_mlpackage("model.safetensors", "config.json", "WhisperTinyEncoder.mlpackage", "WhisperTinyDecoder.mlpackage")
```
나름 효율적으로 해본다고 열심히 encoder decoder도 분리해서 추출했다.

mlpackage 확장자로 그대로 사용해도 xcode 가 자동으로 컴파일 하기 때문에 문제는 없지만 path를 입력하는 과정에서 너무나 헷갈려서 사전에 컴파일러로 mlmodelc 형식으로 변환시켰다.(혹시 할 생각 있는 사람은 mlmodelc 확장자로 사용하는것을 추천한다.)

그리고 모델다운로드, 모델 로드, transcribe, 다국어 지원, 번역, 자막형식 파일지원, 타임스탬프 등등 수많은 기능들을 구현한다고 2주정도 꼬박 만들었던 것 같다. 하지만 결과적으로 실패했다.

## 실패이유 분석
기능 구현 자체는 문제없이 했었던 것 같다. 아마도(?) 하지만 end to end test를 돌리면 무한으로 반복되는 모델 로드 문제를 직면했고 이것또한 해결했지만 tonken화 된 text를 timestamp 와 함께 출력해서 변환하는 과정이 도무지 되질 않아 자꾸 빈 텍스트만 뽑아내기를 삼사일을 보냈던 것 같다.
결과적으로 첫번째 원인인 AI자체의 구동 방식의 이해 부족에 있었다. 사실 파이썬 코드도 제대로 알지 못하고 알음알음 대충 느낌으로 분석했던지라 어찌어찌 구색은 갖췄지만 제대로된 방식인지 나도 알수가 없었다. 두번째 coreML에 대한 이해가 부족했다 coreML 은 처음 사용해보았기 때문에 구동 방식이나 메소드들이 모두 생소했다.
그리고 최신 버전의 swiftTest나 SwiftData 등 사전 지식이 부족한 부분들이 너무나도 많아서 어려움을 겪었다. 계속해서 공부하고 연구하면 어찌어찌 완성은 할 수도 있었겠지만(확신은 못함 ㅠㅠ ) 애초에 목표는 앱을 만드는것이었고 이미 concurrency 등 swift에 맞게 최적화를 마친 패키지를 찾았기때문에 그냥 패키지를 사용하기로 했다.

현실적으로 AI구조에대해 깊이 공부하는것은 쉽지 않겠지만 AI 시대에 CoreML은 앞으로도 마주칠 일이 있을것 같아 더 깊숙히 연구해보고자 한다.

혹시라도 해당 AI에 관심이 있다면 [OpenAI/Whisper](https://github.com/openai/whisper)레포를 살펴보는것도 좋을것 같다. 한국분이 개발담당이신듯? 이것도 신기...
