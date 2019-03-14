---
layout: post
title:  "Generative Adversarial Networks"
author: tigersden
date: 2019-01-10 23:20
tags: [machine-learning, deep-learning, neural-networks]
---

# 서론

2014년, 머신 러닝 분야의 가장 유서깊고 권위 있는 학회인 NIPS에 'Generative Adversarial Networks'라는 한 편의 논문이 발표되었습니다. 이름만으로는 굉장히 난해해 보이는 이 논문의 제목은 한국어로 '생성 적대 망' 정도로 해석할 수 있겠는데요.

이 논문은 기존의 머신 러닝 기술이 다소 약한 모습을 보였던 Generative 모델의 성능을 혁신적인 끌어올린 흥미로운 아이디어를 담고 있었습니다. 당시에는 한계점과 단점 또한 명확한 모델이라는 지적도 있었지만, 점차 단점을 개선한 후속 논문이 나오며 현재는 컴퓨터 비전 분야의 완전한 대세로 자리잡게 되었습니다.

# 핵심 아이디어

Generative Adversarial Networks(이하 GAN)은 특이하게도 두 종류의 네트워크를 가지고 작동합니다. 이름 속의 'Adversarial'(적대적인)이라는 키워드를 통해 우리는 이 두 종류의 네트워크가 어떠한 방식으로든지 서로 대립하게 될것이라는 사실을 유추해 볼 수 있죠.

각각의 네트워크는 'Generator', 'Discriminator'라는 이름이 붙어 있는데, 영어 사전을 뒤져보면 각각 '생성자', '판별자' 정도로 번역을 할 수 있습니다. 조합해보면 무언가를 '생성'하는 네트워크와, 그 결과물을 '판별'하는 네트워크의 서로 간 대립을 통하여 전체 네트워크가 점점 발전을 하게 될 것이라고 추측해 볼 수 있습니다.

논문에서 이것을 보다 쉽게 이해하도록 하기 위한 예시가 등장하는데요. Generator를 지폐 위조범(counterfeiters), Discriminator를 경찰, 네트워크가 생성하는 결과물을 지폐에 비유하였습니다.

![출처 : https://medium.com/@ageitgey/abusing-generative-adversarial-networks-to-make-8-bit-pixel-art-e45d9b96cee7](/assets/images/generative-adversarial-networks/GAN_G.png)

지폐 위조범은 경찰을 속일 수 있는 (상대적으로) 고품질의 위조지폐를 만들어 내려고 시도하며,

![출처 : https://medium.com/@ageitgey/abusing-generative-adversarial-networks-to-make-8-bit-pixel-art-e45d9b96cee7](/assets/images/generative-adversarial-networks/GAN_D.png)

경찰은 위조범의 위조지폐를 진짜 지폐와 구분하려고 시도합니다.

이러한 지속적인 경쟁은 양쪽(지폐 위조범/경찰) 모두의 실력을 점차 발전시키고, 많은 시간이 지난 후에는 위조지폐가 더 이상 진짜 지폐와 구분이 불가능할 정도까지 발전하게 된다는 아이디어입니다. 물론 현실에서는 불가능한 이야기입니다.

# GAN의 학습 과정

앞서 GAN이 어떤 방식으로 동작하는지 비유를 통해 간단하게 알아보았는데요. 본격적으로 GAN이 실제 데이터를 모방해내는 과정을 통해 GAN의 학습 방식에 대하여 자세히 알아보겠습니다.

![GAN으로 사람의 얼굴을 만들어내는 경우](/assets/images/generative-adversarial-networks/GAN_diagram.png)

이 그림은 GAN으로 사람의 흑백 얼굴을 만들어내는 경우를 간단하게 도식화한 것입니다. 그림에서 왼쪽 절차는 진짜 사람의 얼굴으로 학습을 하는 경우, 오른쪽 절차는 가짜 사람의 얼굴로 학습을 하는 경우를 나타냅니다.

여기서 $D$(=Discriminator)는 어떤 얼굴 이미지를 입력으로 받아 이 얼굴이 '진짜 얼굴'일 확률을 출력으로 내뱉는 함수이며, $G$(=Generator)는 랜덤한 노이즈들을 입력으로 받아, 사람의 얼굴을 모방한 가짜 얼굴 이미지를 출력으로 뱉는 함수입니다.

여기서 $G$는 왜 뜬금없이 노이즈를 입력으로 받지? 하는 의문이 생길 수도 있을텐데요. 노이즈 값에 큰 의미가 있다기 보다는 매번 네트워크에 다른 초기값을 주어 매번 다른 얼굴 데이터가 나오도록 하기 위한 것이라고 보시면 됩니다. 프로그래밍을 할때 매번 다른 랜덤 초기값을 얻기 위해서는 랜덤 함수에 대한 시드 값을 항상 다르게 주는 것과 비슷하다고 할 수 있겠습니다.

a. 진짜 데이터에 대한 학습
우선 진짜 데이터의 distribution인 $x$로부터 랜덤하게 데이터 추출합니다. 추출한 데이터를 판별 함수인 $D$에 넣었을 때, $D$는 이 데이터들에 대해서는 1.0을 출력하도록 노력합니다. '노력'이라고 추상적인 단어로 표현했지만 사실은 loss function에 $D$의 출력값과 1.0을 넣고, 복잡한 back-propagation 과정을 통하여 모델 $D$의 파라미터들을 튜닝하는 과정이 되겠지요. 이 과정에서는 G없이 $D$만 등장하여 학습을 진행하기 때문에, 일반적인 classification 기능을 수행하는 Neural Network의 학습 과정와 차이가 없습니다.

b. 위조 데이터에 대한 학습
랜덤 노이즈 벡터를 input으로 받은 네트워크 $G$는 가짜 얼굴 이미지를 만들어내게 됩니다. 이 가짜 이미지들은 이제 모델 $D$의 심판을 통해 진위 여부를 판별받게 되는데요. 여기부터 GAN의 본질인 '대립'이 본격적으로 나타납니다. Discriminator는 이 '가짜 이미지'들을 제대로 걸러내고 싶어 하기 때문에 $D$에서 0.0을 출력하기를 원하고, 반대로 Generator는 $D$가 속아넘어가기를 바라기 때문에 $D$에서 1.0이 나왔으면 합니다.

간단하게 요약하자면,

1. $D$는 다음에 가짜 얼굴들을 더 잘 걸러내기 위해 이 데이터들에 대해서는 $0.0$을 출력하도록 자신을 변화하며 노력합니다.
2. $G$는 다음에 가짜 얼굴들로 $D$를 더 잘 속이기 위해, $D$가 이 데이터들에 대해 $1.0$을 출력하는 방향으로 자신을 변화시키도록 노력합니다.

일반적으로 '판별'보다는 '위조'가 더 어렵기 때문에 $G$보다는 $D$가 더 빨리 학습되는데요. 그래서 iteration이 계속 반복되면 처음에는 조악한 수준의 가짜 얼굴만을 찍어내던 Generator는 점점 더 가짜 얼굴을 꽤나 잘 판별하는 Discriminator를 속이는 방향으로 진화를 거듭하게 되고, 결국에는 실제 얼굴들과 구분이 힘들 정도의 그럴듯한 가짜 얼굴들을 만들어 내는 수준까지 이르게 됩니다.

# Loss Function

이 학습 과정을 수행하기 위한 GAN의 Loss function은 다음과 같습니다. $G$와 $D$가 minimax 게임을 진행한다고 생각하고 짜여진 수식인데요. 복잡해 보이지만 찬찬히 뜯어보면 생각보다 어렵지 않습니다.

![GAN의 Loss Function](/assets/images/generative-adversarial-networks/GAN_loss.png)

우선 이 loss function을 전체적으로 보면, $\underset{G}{min}$와 $\underset{D}{max}$라는 표현 방식에서 알 수 있듯이 Generator는 전체 식인 $V(D, G)$를 최소로 만들려고 하며, Discriminator는 전체 식 $V(D, G)$를 최대로 만들려고 노력합니다.

식의 첫 번째 항은 $D$가 진짜 데이터를 판별하는 것에 관한 loss입니다. $x$가 $p_{data}$ distribution에서 추출한 데이터일때, $D$가 이상적인 경우 $D(x)$는 $1.0$이 나오고 $log\ D(x)$는 $0$이 나와야하는 것이죠. 앞서 설명하였듯이 이 과정에서는 Generator가 영향을 주지 않기 때문에 수식 상에도 $G$가 등장하지 않는 것을 볼 수 있습니다.

두 번째 항은 GAN의 핵심인 가짜 데이터를 판별하는 것에 관한 loss입니다. 우선 $G(z)$는 아까도 보았듯이 Generator 네트워크에 랜덤 노이즈를 넣어 만들어낸 가짜 데이터입니다. 그래서 $D$의 입장에서 이상적인 경우는 $D(G(z))=0.0$가 되며, $log\ (1-D(G(z)))$는 $0$인 경우입니다. 반대로 $G$의 입장에서는 D를 속이기 위해, 즉 $D(G(z))=1.0$을 만들어야 하기 때문에, $log(1-D(G(Z))$)를 $-\infty$로 만들려고 합니다.

이렇게 두 네트워크가 loss function을 가지고 첨예한 줄다리기 과정을 거친 끝에야, 결국 Generator는 그럴듯한 이미지를 만들어 낸다는 소기의 적을 달성할 수 있게 되는 것입니다.

# Pseudo-code

의사코드를 보며 더 자세한 구현에 대해 살펴보겠습니다.

![GAN의 기본 알고리즘을 나타낸 pseudo-code](/assets/images/generative-adversarial-networks/GAN_algorithm.png)

우선 가장 밖의 반복문이 각각의 iteration을 나타낸다는 사실은 간단하게 알아볼 수 있습니다. 더 아래쪽을 보면 특이하게도 한 iteration당 'Discriminator를 $k$번 학습 후, Generator를 1번 학습'이라는 과정을 거치는 것을 볼 수 있는데요.

이는 본문에서도 나오지만 Discriminator를 먼저 빠르게 일정 수준 이상으로 학습시켜, Discriminator가 제대로 된 판별자 역할을 하여 Generator를 올바른 길(?)로 이끌기 위함 입니다. Generator와 Discriminator가 서로 경쟁하며 발전한다는 특성 상, 어느 한쪽은 제대로 된 작동을 하여야 전체적인 학습 과정이 정상 궤도에 오를 수가 있는데, 아무래도 $G$와 $D$중에 학습이 쉬운 쪽은 $D$이기 때문입니다. Generator가 초반에 출력하는 결과물은 대체로 형편없기 마련이고, 이를 통해 간단하게 진짜/가짜 데이터을 판별하는 것 뿐이니까요.

다시 코드에서, $D$의 학습 과정에서는 우선 $m$개의 노이즈 샘플을 추출하는 것으로부터 시작합니다. 그리고 실제 데이터 distribution으로부터 같은 갯수만큼 데이터를 선별하는데요. 이렇게 고른 진짜 데이터 $m$개 / 노이즈로부터 만든 가짜 데이터 $m$개를 가지고 Discriminator의 학습을 진행합니다. $G$의 학습 과정에서는 진짜 데이터는 사용하지 않습니다. $m$개의 노이즈 샘플을 추출하여 가짜 데이터들을 생성하고, 이를 바탕으로 $D$를 속여 넘기기 위한 방향으로 $G$를 학습하게 됩니다.

# 실험 결과

![GAN의 가짜 이미지 생성 결과](/assets/images/generative-adversarial-networks/GAN_result.png)

MNIST 손글씨 데이터 셋과, 사람 얼굴 데이터 셋(이름을 모르겠네요)을 이용하여 GAN을 학습한 결과물입니다. 보다시피 MNIST 데이터 셋의 경우는 꽤 리얼한 모방 이미지 생성을 보여주었고, 오른쪽의 얼굴 데이터 셋의 경우에도 흐린 이미지들이 일부 있긴 하지만 전체적으로 그럴듯한 이미지 생성 능력을 보여주고 있습니다.

# 결론

GAN은 기존 Generative 모델의 문제점을 굉장히 창의적인 방법으로 풀어냈습니다. 하지만 단점 또한 명확하게 존재하였는데요. real data의 distribution이 multi-modal일 경우 전체 분포를 골고루 모방하는 대신 한 종류의 결과물만을 뱉어내는 mode-collapse 현상이 발생한다거나, 학습이 불안정하여 해상도를 올리는 것이 매우 어렵다거나 하는 등 치명적인 단점이 몇 가지 있습니다.

하지만 이후에 GAN을 발전시킨 후속 논문들이 그야말로 쏟아져 나오면서 많은 단점들은 해결이 된 상태이며, 서론에 적었듯이 현재는 이미지 생성 분야의 대세가 되었습니다. 다음 글에서는 GAN의 본격적인 부흥기를 가져왔다고 평가받는 DCGAN에 관하여 알아보겠습니다.

# PyTorch Implementation

공부하면서 간단하게 만들어보았던 toy code 입니다.
PyTorch를 처음 배울때 작성한거라 코드 수준이 조악한 점 양해 부탁드립니다.
https://github.com/tigersdennn/GAN_toy_code