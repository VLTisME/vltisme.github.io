---
title: Intuition in Computer Vision
date: 2025-04-02 15:00:00 +0700
categories: [AI, DL]
tags: [computer vision, intuition]     # TAG names should always be lowercase
authors: [tuan]

description: "This is exactly how I visualize and understand things so easily (or not)."
toc: false
comments: true

---




This is only for R-CNN related stuff: (learn the abstract first - understand the architecture, we use X, Y , Z in here,… then what does X do, Y do, Z do? → then deepdive into theory - how is it structured? how does it do so well? why does it work? → then into Code, wats the workflow? where should i study fist? which part of the code should be read first?

Note:

- Why R-CNN is slow? → Due to 3 independent modules: it must first propose regions, then each region is fed into CNN to extract module
- How Fast R-CNN optimizes it? It use only one single model instead of a pipeline like above.

Understand Fast-RCNN:

- Ban đầu sẽ feed vào một tấm ảnh, tấm ảnh đó, thay vì áp dụng region proposal như R-CNN để ra một mớ regions thì nó lại được truyền qua một Deep CNN để trích xuất ra feature map. Vậy làm sao để biết vị trí của một region trên feature map đó? chúng ta sẽ dùng phép chiếu Rol để “tương đối” được vị trí của region đó trên ảnh gốc. sau đó truyền cả kết quả của rol projection và feature map (Lưu ý là truyền duy nhất 1 rol vào thôi, ko truyền cả chùm) vào các rol pooling layers , rồi vào FCs để thu được các rol feature vectors. Sau đó các rol feature vectors sẽ đi qua 2 nhánh. 1 Nhánh giúp xác định phân phối xác suất theo các class của 1 vùng quan tâm RoI thông qua hàm softmax và nhánh còn xác định tọa độ của bounding box thông qua hồi qui các offsets.

Faster-RCNN:

- Nó thực chất là sự kết hợp của hai modules riêng lẻ: Fast-RCNN và RPN (region proposal network) - RPN là một CNN để đề xuất các vùng và loại đối tượng cần xem xét trong vùng.

tức là cũng như Fast-RCNN ở trên, Faster RCNN nó sẽ tối ưu ở cái chỗ Rol projection (nó sẽ ko tạo rol projection trên featuremap), nó sẽ dùng RPN để trích xuất ít vùng hơn nhiều từ những cái feature maps sau khi input flow qua DCNN, rồi số vùng đề xuất được sẽ cùng được feed vào rol pooling layers với feature maps từ DCNN. và cũng khác ở chỗ là ở Fast RCNN, nó ko tạo Rol Projection mà dùng RPN để tạo ra regions từ output của DCNN luôn. DCNN thường là VGG-16, ResNet,…

ồ và RPN nó propose regions và những regions đó là bounding box luôn rồi nên ta chỉ cần dùng feature map, đưa feature map vào một classifier là xong, ko cần regressor để tìm bounding box.

Mask-RCNN:

cái việc dự đoán nhãn của region đó và masking nó sẽ được chạy song song!

khi mà qua backbone (resnet + FPN) á, thì nó cho ra cái feature map ok, rồi RPN nó sẽ trích xuất regions ra từ feature map đó, mà các regions đó chưa biết vị trí trong ảnh gốc là ntn nên sẽ đưa cái feature map và regions đó vào RoIAlign để ước tính vị trí của regions đó trong ảnh gốc, và RolAlign còn làm cả việc extract features cho mỗi region được proposed → hình như ko phải, mà là vì regions proposed thì có size khác nhau nên làm sao mà feed vào NN sau đó một cách hiệu quả được? nên RoIAlign mới đưa chúng nó về fixed size rồi mới feed vào NN (Trong paper gọi vấn đề đó là “misalignment” đó) → Đúng, còn việc loại bỏ regions ko quan trọng thì đã làm ở RPN

→ RPN nó đưa ra bounding box (aka RoI) với lại objectness score, xong rồi lấy kết quả đó, kết hợp với feature maps (nhớ là feature maps chứ ko phải original image), kết hợp ở đây ý mình là nó chiếu cái vùng đề xuất lên feature map để lấy thông tin cái vùng đó ở feature map rồi feed vào thằng thằng RoIAlign, rồi với mỗi vùng đề xuất trích ra từ feature map RoIAlign sẽ dùng Bilinear interpolation để tái hiện lại gần đúng… rồi nó normalizes/ aligns all kết quả sau khi bilinear interpolation về cùng một size rồi feed vào FCs ở sau.

→ RolAlign: first it takes the RoI (the bounding box) and extract info from the feature map of that RoI, then it divide the RoI into smaller bins (7x7 for classification and 14x14 for masking?), it divides into floating points coordinate (e.g: RoI size is 40x40 then each bin will cover ~5.7 x 5.7 pixels on the feature map) → then it uses Bilinear Interpolation with values in the feature map from backbone.

question: why would resizing into 7x7 work well in practice (e.g if 40x40 contains a giraffe, then doesnt 7x7 contain only a part of that giraffe and make the model predict poorly?): 

- The input to RoIAlign is not the raw image but the **feature map** produced by the backbone (e.g., ResNet, FPN). These feature maps already encode high-level semantic information about the object (e.g., "giraffe") in a **spatially compressed form**.
- For example, if the backbone downsamples the image by a factor of 16, a 40×40 region on the feature map corresponds to a much larger region (640×640) in the original image. The feature map has already condensed the visual information into a smaller, more abstract representation.

**→ this means your understanding about conv is not corret, a kernel size does relate to the region, the size it looks, but you forgot that even if a kernel size of just 5x5 can capture the whole 40x40 giraffe thanks to sliding the kernel over the picture and outputs a lot of channels, each channel preserves informations about the giraffe (ex: its ears, its body, its heads,…) and the network can “combine” these spatial information into bigger one to reshape the giraffe! so interesting! And also, small kernel size can capture smaller object like a squirrel on the giraffe’s back!!! kernel size not only able to capture large or small object but can also act as a “scissor” to cut the image into smaller parts, each part retains information, and then reconstruct it later!”**

question: larger bin will capture large region but why do we also use large bin to mask very small object like a distant bird?:

- You're absolutely right to ask this! If a distant bird (a very small object) corresponds to a small region on the feature map, then dividing it into relatively **large bins** (as part of a fixed `7×7` or `14×14` grid) might seem counterintuitive.
- oh simply just big region or small region will be divided equally… by 14? hm?

hm hãy sẵn sàng để tưởng tượng điều thú vị này mình nghĩ:

- Xét một CNN, thường thì các kernel size ban đầu sẽ lớn xong nhỏ dần. Để làm gì? kernel size lớn thì nó capture được một vùng ảnh rộng hơn… → sau đó lan truyền và lan truyền thông tin qua các layers sau… và lan truyền nhiều quá rồi thì hơi sợ mất thông tin của vùng ảnh đó do exploding hoặc vanishing gradient descent nên sẽ có residual connections → xong rồi để xét một vùng ảnh nhỏ hơn liệu có gì đặc biệt đáng chú ý không (ví dụ vùng lớn có con hưu cao cổ, vùng nhỏ có con khỉ ngồi trên lưng hưu cao cổ) thì ta tiếp tục introduce smaller kernel size đối cho các lớp sau để cái kernel size đó tiếp tục capture một vùng nhỏ của cái vùng ảnh lớn ban đầu! → deeper network + techniques = capture more specific context. - Thật ra cũng không hẳn =)) ko cần phải kernel size càng ngày càng nhỏ, ví dụ như nó = 2 miết thì vẫn được mà =)).

→ hỏi chatgpt về intuition của những kĩ thuật trong CV đi, thú vị lắm. VD như batchnorm thì hữu ích khi mà có một feature quan trọng nhưng dao động trong khoảng [0-1] còn có thằng dao động từ [0-10000].

Thì những thằng trên là OK về mặt Object Detection rồi, nâng cao hơn nữa về object detection sẽ là về YOLO family… nhưng mà mình sắp tới thi về “instance segmentation” nên… tiếp theo sẽ không học YOLO mà là Mask R-CNN 😲 vừa có khả năng object detection vừa có khả năng masking.

**thú thật**, mặc dù đọc paper, một thứ cực kì chuyên sâu và chi tiết, nhưng mình vẫn rất dễ hiểu và nắm chắc kiến thức vì- mình đã học nó một cách abstract rồi :))) mình đã đọc qua các blog overview về nó và biết nó có gì, nó làm được gì, nó tối ưu cái gì,… còn đọc paper chỉ là để biết nó làm những điều đó như thế nào thôi, và hệ thống lại kiến thức.

tóm tắt sơ lại workflow học của mình: Thi về Instance segmentation → hỏi chatGPT về những model tốt nhất → lựa ra 1 model để học → hỏi chatgpt về prequesites cần có để học model đó → … ok bh là đối với từng prequisite, hoặc là model → hỏi chatgpt đưa ra roadmap để mình học về thứ đó → xem thử model đó có những thành phần gì và công dụng từng thành phần, nhờ chatgpt nói về workflow của model đó, tự mình giải thích và tưởng tượng lại workflow → deepdive into each component: đào sâu vào từng thành phần, hiểu được công thức của nó, intuition đằng sau nó, làm sao nó có công dụng như vậy… tự giải thích lại (đoạn này nên ghi lại tất tần tật components của model lớn theo như workflow chatgpt nói, rồi dựa vào những gì đã ghi để nhờ chatgpt giải thích từng phần) → sau khi đã hiểu rõ từng thành phần của một model lớn rồi thì quay lại đọc chuyên sâu paper của model lớn, lúc này đọc đến đâu cũng hiểu đến đó vì các khái niệm trong paper lớn mình đã học hết rồi! → xong xuôi hết rồi thì tự tổng hợp lại tất tần tật từ đầu đến cuối lại một lần nữa: kiến trúc của model là gì? thành phần này có chức năng gì? tại sao nó làm được như vậy? workflow của model đó là như thế nào. 

Tương tự cho coding phase: hiểu workflow của code trước đã, rồi xem thử cần phải học những gì (prequesites) để hiểu code? rồi mới đọc kĩ từng phần của code, rồi code lại!.

“Tell me the workflow of model X, and each of its component’s usage, the intution behind it (overview and abstract part). Then provide me a roadmap to learn these components first, in order from what need to study first to harder parts. Then learn each component, for each component, ask ChatGPT besides mathematical knowledge, what are the prequesites to understand that component? Then continue divide and conquer,… once the smaller part is done, back to the larger part, then larger… largest! (breakdown the big picture into smaller ones, then continue breaking down the smaller into even smaller ones, then study from the smallest, to understand the second smallest,… to the big picture!).

**Yay… khám phá ra cách học đỉnh cao học siêu nhanh! Chỉ trong 30p mà mình hiểu hết 3 architecturés OMG! ra đây là sức mạnh của CÁCH HỌC → đó là: phải biết học gì trước (ask ChatGPT) → học cái rộng, bao quát, trừu tượng để nắm được workflow của nó, cách nó hoạt động (đọc overview về nó, biết nó có gì xong biết nó dùng để làm gì, what does it use and what does it do?) → deepdive into từng phần (làm sao nó làm được như vậy? công thức của nó là gì? How does it do that?) (áp dụng cho cả việc đọc paper or học một kiến thức mới và coding phase).**

Continue Mask-RCNN paper reading (remember: from abstract, overview to deepdive into each part then back to the big picture) → Coding phase (remember: understand the workflow first then understand each file later). Ôn kĩ lại intuition từng li từng tí Mask RCNN (như là ý nghĩa của RPN, RoIAlign, Backbone architecture,…) và các thao tác khác trong boxchat intuition & life với sider. Viết blog về Mask-RCNN thật kĩ càng và dễ hiểu theo kiểu Intuition & học kiểu mới của mình để cho nhóm AIC của mình đọc. Có thể đọc nhanh DETR

“You must be the one with the model to understand the power of it. Once fully understand it, it is like understanding yourself” - “To create a new model, think like yourself is the current models, all the techniques you know, and specify the problem, what do you have to solve it, what do you lack of, what is not good enough, what if we get rid of this, add something new, change it to something we have never seen…”?. 

