# iOS Exposition Universelle Application Project
### 테이블뷰를 활용한 다양한 국가의 문화유산 리스트를 구현한 앱 프로젝트
[Ground Rule](https://github.com/GREENOVER/ios-exposition-universelle/blob/main/GroundRule.md)
***
#### What I learned✍️
- UItableView (Dynamic / Static / Custom)
- TableCell
- InterfaceOrientationMask
- JSON
- Decode & Encode
- NumberFormatter
- ViewLifeCycle
- automaticDimension
- Codable
- CodingKey

#### What have I done🧑🏻‍💻
- 테이블뷰의 종류에 대해 학습하고 커스텀하여 테이블 뷰를 상황에 맞게 구현하였다.
- 셀을 통해 데이터를 담아주었다.
- 앱 딜리게이트에서 가로세로 기기 회전 변경을 지원하였다.
- 디코드와 인코드를 학습하여 프로젝트에서 디코드를 통해 JSON 데이터를 변환하여 사용하였다.
- 숫자 형태를 커스터마이징하여 원하는 값으로 변환하였다.
- 뷰의 생명주기를 학습하고 각 뷰의 이동 및 상태에 따라 코드를 구현하였다.
- 테이블뷰의 높이를 자동으로 맞게 구현하였다.
- 테이블뷰의 셀이 클릭되었을때 기능을 구현하였다.
- 디코딩과 실패 시 디코딩 에러를 세분화하여 구현하였다.
- 다양한 뷰에서의 테이블에 맞는 각 모델을 설계하고 정의하였다.
- 모델 설계 시 Codable 프로토콜을 채택하여 JSON 데이터에 맞게 구현하였다.

#### Trouble Shooting👨‍🔧
- 문제점 (1)
  - JSON 데이터를 디코딩 시 오류가 날때 직관적으로 해당 오류의 종류를 알 수 없는 문제
- 원인
  - 다양한 에러에 대한 처리를 해주지 않았기에 디버깅 메시지를 통해 찾을 수 밖에 없는 문제가 발생하여 가독성이 떨어지고 개발 효율이 낮아졌다.
- 해결방안
  - 아래와 같이 디코딩에러에 관해 세부적으로 나누어 에러 발생 시 어떤 에러인지 description 얼럿을 나타내도록 구현하였다.
  ```swift
  do {
            let decodeData = try jsonDecoder.decode(T.self, from: dataAsset.data)
            return decodeData
        } catch DecodingError.dataCorrupted(let context) {
            print("데이터가 손상되었거나 유효하지 않습니다.")
            print(context.codingPath, context.debugDescription, context.underlyingError ?? "" , separator: "\n")
            showAlertMessage("데이터가 손상되었거나 유효하지 않습니다.")
        } catch DecodingError.keyNotFound(let codingkey, let context) {
            print("주어진 키를 찾을수 없습니다.")
            print(codingkey.intValue ?? Optional(nil)! , codingkey.stringValue , context.codingPath, context.debugDescription, context.underlyingError ?? "" , separator: "\n")
            showAlertMessage("주어진 키를 찾을수 없습니다.")
        } catch DecodingError.typeMismatch(let type, let context) {
            print("주어진 타입과 일치하지 않습니다.")
            print(type.self , context.codingPath, context.debugDescription, context.underlyingError ?? "" , separator: "\n")
            showAlertMessage("주어진 타입과 일치하지 않습니다.")
        } catch DecodingError.valueNotFound(let type, let context) {
            print("예상하지 않은 null 값이 발견되었습니다.")
            print(type.self , context.codingPath, context.debugDescription, context.underlyingError ?? "" , separator: "\n")
            showAlertMessage("예상하지 않은 null 값이 발견되었습니다.")
        } catch {
            print("그외 에러가 발생했습니다.")
            showAlertMessage("그외 에러가 발생했습니다.")
        }
  ```

- 문제점 (2)
  - 각 화면마다 회전 설정에 대해 동일하게 적용되는 문제
- 원인
  - AppDelegate에서 화면 회전 설정에 대해 all로 설정되어 모든 화면에서 동일한 설정이 적용되는 문제
- 해결방안
  - 각 화면에서 가로모드를 지원하지 않고 싶을때를 위해 우선 AppDelegate의 딜리게이트를 인스턴스로 생성하여 뷰가 사라지거나 나타날때 회전을 다르게 적용하였다.
  ```swift
  private let appDelegate = UIApplication.shared.delegate as! AppDelegate
  override func viewWillAppear(_ animated: Bool) {
        self.navigationController?.navigationBar.isHidden = true
        
        appDelegate.shouldSupportAllOrientation = UIInterfaceOrientationMask.portrait
  }
  override func viewWillDisappear(_ animated: Bool) {
      self.navigationController?.navigationBar.isHidden = false
      
      appDelegate.shouldSupportAllOrientation = UIInterfaceOrientationMask.all
  }
  ```
- 문제점 (3)
  - 테이블 뷰의 높이가 담긴 데이터에 부합되지 않는 문제
- 원인
  - 테이블 뷰의 높이에 대한 자동설정이 없어 지정해준 크기만큼 테이블뷰가 나타나게되어 발생
- 해결방안
  - 테이블 뷰의 높이를 오토매틱하게 변경해주는 메서드를 오버라이드하여 추가
  ```swift
  override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
      let height = UITableView.automaticDimension
        
      return height
  }
  ```
- 문제점 (4)
  - 사용자가 알아차리는 유의미한 문제는 아니지만 viewWillAppear시 애니메이션 효과에 대해 false로 초기에 주었는데 앱 뷰 전환이 느리다면 어색할것같다.
  ```swift
  override func viewWillAppear(_ animated: Bool) {
      super.viewWillAppear(false)
  }
  ```
- 원인
  - 애니메이션 효과의 Bool값으로 true / false가 들어가서 애니메이션이 나오는지 마는지를 컨트롤 할 수 있는데 이 값을 false로 주게되면 애니메이션을 하지 않아 발생한다.
- 해결방안
  - 처음에는 애니메이션 값을 true로 다 주는것이 현명하지 않을까 싶었는데 시스템에서 정한 값을 주도록 아래와 같이 변경하였다.
  ```swift
  override func viewWillAppear(_ animated: Bool) {
      super.viewWillAppear(animated)
  }
  ```
  파라미터의 인자를 넘겨주는 방식으로 시스템이 정하게 하였으며 궁금해서 디버깅을 통해 기본 애니메이션 설정값을 보았는데 true이다.
  <img width="924" alt="스크린샷 2021-04-27 오후 1 46 11" src="https://user-images.githubusercontent.com/72292617/116186591-ef63ed80-a75e-11eb-900f-edd19f2647c5.png">

#### Thinking Point🤔
- 고민점 (1)
  - "JSON 파일에서 넘어오는 스네이크케이스 프로퍼티명을 Swift에 맞는 카멜케이스로 변경해줄 수 있을까요?"
- 원인 및 대책
  - JSON 데이터에 대한 모델 타입을 설계할때 CodingKeys 열겨형을 활용하여 원하는 네이밍으로 변경할 수 있다.
  ```swift
  enum CodingKeys: String, CodingKey {
        case name
        case imageName = "image_name"
        case shortDescriptions = "short_desc"
        case descriptions = "desc"
    }
  ```
- 고민점 (2)
  - "문자열로 된 이미지 파일명을 이용해 UIImage로 바꿔 활용할 수 있는 방법이 있을까요?"
- 원인 및 대책
  - 모델을 설계할때 우선 이미지를 문자열로 받고 연산 프로퍼티를 활용해 UIImage를 반환해주어 사용할 수 있다.
  ```swift
  struct KoreanEntries: Codable {
    let name: String
    let imageName: String
    let shortDescriptions: String
    let descriptions: String
    
    var image: UIImage? {
        return UIImage(named: self.imageName)
    }
  }
  ```
  이렇게 해준다면 뷰 컨트롤러에서 변환해줘야하는 기능을 타입에서 해줌으로써 기능을 더 분리할 수 있다.
- 고민점 (3)
  - "viewDidLoad 메서드가 꼭 필요할까요?"
- 원인 및 대책
  - viewDidLoad 메서드가 당장 필요하지 않다면 없애는것이 더 좋다. 마지막 화면으로 전환될때는 viewDidLoad에서 동작해주는 부분이 없기에 아래와 같이 viewWillAppear에서 데이터 값 어사인을 구현해주었다.
  ```swift
  class KoreanEntryViewController: UITableViewController {
    
    var fetchData: KoreanEntries?
    
    @IBOutlet weak var koreanEntryImageView: UIImageView!
    @IBOutlet weak var koreanEntryDescriptionsTextLabel: UILabel!
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(false)
        
        assignData()
    }
    
    func assignData() {
        guard let fetchData = self.fetchData else { return }

        navigationItem.title = fetchData.name
        koreanEntryImageView.image = fetchData.image
        koreanEntryDescriptionsTextLabel.text = fetchData.descriptions
    }
    
    override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        let height = UITableView.automaticDimension
        
        return height
    }
  }
  ```
- 고민점 (4)
  - "guard let 구문을 연달아 작성 시 아래와 같이 작성하는것이 좋은 가독성일까요?"
  ```swift
  guard let fetchName = koreanEntryNameData,
        let fetchImage = koreanEntryImageData,
        let fetchDescriptions = koreanEntryDescriptionsData else { return }
  ```
- 원인 및 대책
  - guard let에 관한 애플 공식문서를 참고하여 개선해보았다.
  ```swift
  guard let fetchName = koreanEntryNameData, let fetchImage = koreanEntryImageData, let fetchDescriptions = koreanEntryDescriptionsData else {
    return
  }
  ```
- 고민점 (5)
  - "화면 회전에 대한 변수를 UIInterfaceOrientationMask 타입으로 설정 할 수는 없을까요?"
  ```swift
  import UIKit
  @main
  class AppDelegate: UIResponder, UIApplicationDelegate {
     var shouldSupportAllOrientation = true
     func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
         if shouldSupportAllOrientation {
             return UIInterfaceOrientationMask.all
         }
         return UIInterfaceOrientationMask.portrait
      }
  ```
- 원인 및 대책
  - 화면 회전에 대해 해당 타입으로 변경하여 앱딜리게이트의 설정 메서드 부분의 코드를 줄였다.'
  ```swift
  import UIKit
  @main
  class AppDelegate: UIResponder, UIApplicationDelegate {
    var shouldSupportAllOrientation: UIInterfaceOrientationMask = .all
    func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
         return self.shouldSupportAllOrientation
     }
  }
  ```


#### InApp📱
![InAppTest](https://user-images.githubusercontent.com/72292617/116182712-b4aa8700-a757-11eb-8bcd-585c2de8e955.gif)
