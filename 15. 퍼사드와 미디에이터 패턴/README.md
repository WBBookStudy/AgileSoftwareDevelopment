# 15. 퍼사드와 미디에이터 패턴
퍼사드는 위로부터 정책을 적용하고, 미디에이어는 아래로부터 정책을 적용한다.


## 퍼사드 패턴
복잡하고 일반적인 인터페이스를 갖는 객체 그룹에 간단한고 구체적인 인터페이스를 제공하고자 할 때 사용

![KakaoTalk_Photo_2024-03-17-15-38-11](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/e45ed896-798d-4de7-8269-9870d5cf1173)
DB클래스가 Application이 java.sql 패키지 안에 내용을 숨기고 있다는것을 주목하자.  
Application의 관점에서 보면, java.sql은 존재하지도 않는다.  

## 미디에이터 패턴
미디에이터패턴도 정책을 적용함.  
퍼사드가 자신의 정책을 가시적이고 강제적인 방식으로 적용하는 반면, 미디에이터는 자신의 정책을 은밀하고 강제적이지 않은 방식으로 적용한다.  

```Java
package utility;

import javax.swing.*;
import javax.swing.event.*;

/**
 이 클래스는 JTextField 하나와 Jlist 하나를 받는다. 이 클래스는 사용자가 JList 에 있는 항목들의 접두어 (prefix)를 JTextField에 입력한다고 가정한다. JrextField 의 현재 접두어와 일치하 JList 의 첫 번째 항목을 자동적으로 선택한다. 만약 JTextField 가 null 이거나 접두어가 JList에 있는 어떤 원소와도 일치하지 않으면, JList 의 선택은 지워진다. 이 객체를 호출하기 위한 빙법은 없다 그냥 생성하고, 잊어 버리면 된다 (하지만 가비지 콜렉션에 의해 없어지도록 놔두지 않도록 한다).
 */

/** 예제
 JTextField t = new JTextField();
 JList l = new JList();

 QuickEntryMediator qem = new QuickEntryMediator(t, l); // 이게 전부임
 */

public class QuickEntryMediator {
  public QuickEntryMediator(JTextField t, JList l) {
    itsTextField = t;
    itsList = l;

    itsTextField.getDocument().addDocumentListener(
      new DocumentListener() {
        public void changedUpdate(DocumentEvent e) {
          textFieldChanged();
        }

        public void insertUpdate(DocumentEvent e) {
          textFieldChanged();
        }

        public void removeUpdate(DocumentEvent e) {
          textFieldChanged();
        }
      } // new DocumentListener
    ); // addDocumentListener
  } // QuickEntryMediator()

  private void textFieldChanged() {
    String prefix = itsTextField.getText();

    if (prefix.length() == 0) {
      itsList.clearSelection();
      return;
    }

    ListModel m = itsList.getModel();
    boolean found = false;
    for (int i = 0; found == false && i < m.getSize(); i++) {
      Object o = m.getElementAt(i);
      String s = o.toString();
      if (s.startsWith(prefix)) {
        itsList.setSelectedValue(o, true);
        found = true;
      }
    }

    if (!found) {
      itsList.clearSelection();
    }
  } // textFieldChanged

  private JTextField itsTextField;
  private JList itsList;

} // class QuickEntryMediator
```
> 이 클래스는 조용히 이면에서 텍스트 입력 필드를 어떤 리스트와 결합한다.  

![KakaoTalk_Photo_2024-03-17-15-48-29](https://github.com/WBBookStudy/AgileSoftwareDevelopment/assets/60125719/152adc99-8bc9-48fa-9062-09ca24967ee7)
JList와 JTextFiled의 사용자는 이 미디에이터가 존재하는지 알지 못한다. 



























