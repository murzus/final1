final1
======

gui+jcp
package guiv2;
/**
 *
 * @author VBykov
 */
import java.awt.*;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Timestamp;
import java.text.ParseException;
import java.text.SimpleDateFormat;
//import java.util.ArrayList;
import java.util.Calendar;
import java.util.GregorianCalendar;
import java.util.Vector;
import javax.swing.*;
import javax.swing.table.AbstractTableModel;
import javax.swing.table.DefaultTableModel;
import javax.swing.text.MaskFormatter;


public final class GUIv2 extends JFrame{

    static private Connection con = null;
    static private PreparedStatement  stmt = null;
    static private ResultSet rs;
    static private String cuid;
    static private String from;
    static private String to;
    static private LimitTextField cuidText = new LimitTextField(5);
    static private JTextArea info= new JTextArea(10,60);
    JFormattedTextField  dateFrom;
    JFormattedTextField  dateTo;
    DefaultTableModel model;
    JScrollPane tableScroll;
     JTable table;
     JPanel gui; 
     JComboBox logsCombo;
     final static String[] logsString= { " ",
                                         "hc_comon" ,
                                         "hc_portal" , 
                                         "hc_gba"
      };
     
    GUIv2 (String s){
         super(s);
         initForm();
    }
     private static boolean checkCon() throws ClassNotFoundException, SQLException{
            Class.forName ("oracle.jdbc.driver.OracleDriver");
            con = java.sql.DriverManager.getConnection("jdbc:oracle:thin:@os-0378:1521:GFO","gfo_obninsk","hjk321");
       return (con!=null);
    }
    
    public static void main(String[] args){
        GUIv2 guIv2 = new GUIv2("expIBS");   
    } 
    
 void initForm() {
        try {
             setMinimumSize(new Dimension(1024,768));
             setDefaultCloseOperation(JFrame.DO_NOTHING_ON_CLOSE);
             addWindowListener(new WindowAdapter(){
              @Override
             public void windowClosing(WindowEvent e){
             int rez1=JOptionPane.showConfirmDialog(rootPane,"Выйти из программы?",
                                                    "Super",JOptionPane.YES_NO_OPTION,JOptionPane.QUESTION_MESSAGE);                                                     
              if(rez1==0){System.exit(0);}    
           }
         });
            final JPanel parametrPan = new JPanel();
            final JPanel parametrs  = new JPanel(new GridLayout(0,2,3,3));
            //final JPanel checkBoxes = new JPanel(new GridLayout(1,0));
      
      JLabel labelFrom =new JLabel("Date From",JLabel.RIGHT);       
      JLabel labelTo =new JLabel("Date To ",JLabel.RIGHT); 
      JLabel cuidLabel =new JLabel("CUID",JLabel.RIGHT); 
      cuidText.setTextLengthLimit(8);
        
     MaskFormatter maskDate;
     maskDate = new MaskFormatter("20##-##-## ##:##:##");
      
      dateFrom=new JFormattedTextField(maskDate);
      dateTo = new JFormattedTextField(maskDate);
     parametrs.add(cuidLabel);
     parametrs.add(cuidText);
     parametrs.add(labelFrom);
     parametrs.add(dateFrom);
     parametrs.add(labelTo);
     parametrs.add(dateTo);
     
       
            Calendar now = new GregorianCalendar();  
            final SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String nowString = format.format(now.getTime());         
            dateTo.setText(nowString);
            
            Calendar prevMonth=new GregorianCalendar();
            prevMonth.add(2, -1);
            String prevMonthString = format.format(prevMonth.getTime());
            dateFrom.setText(prevMonthString);
            
     JCheckBox cpregCheckBox = new JCheckBox("CPREG");
     JCheckBox cifCheckBox = new JCheckBox("CIF");
     JCheckBox mfmsCheckBox = new JCheckBox("MFMS");
     
     
     parametrs.add(new JLabel("Service History",JLabel.RIGHT));
     parametrs.add(cpregCheckBox);
     parametrs.add(new JLabel("Data from CIF",JLabel.RIGHT));
     parametrs.add(cifCheckBox);
     parametrs.add(new JLabel("SMS(#ph GFO)",JLabel.RIGHT));
     parametrs.add(mfmsCheckBox);
        
    final JButton check= new JButton("Test connect");
    parametrs.add(check);
    parametrPan.add(parametrs);
            
            
            
 check.addActionListener(new java.awt.event.ActionListener() {
    @Override
 public void actionPerformed(java.awt.event.ActionEvent evt) {
   try {                 
    if(checkCon()){
           final JButton go= new JButton();
           go.setText("GO!");
      go.addActionListener(new java.awt.event.ActionListener() {
         @Override
       public void actionPerformed(java.awt.event.ActionEvent evt) {
          if(cuidText.getText().trim().length()<4) {
                      JOptionPane.showMessageDialog(null,"Введите корректный cuid",
                                                "Ошибка",JOptionPane.ERROR_MESSAGE);
          } else{
                     cuid=cuidText.getText().trim();
                     initData();
            }
      }
     });
           parametrs.add(go);
           pack();
           check.setVisible(false);
       }
      }catch ( SQLException | ClassNotFoundException ex) {
                 JOptionPane.showMessageDialog(null,ex,"Error",JOptionPane.ERROR_MESSAGE);
            }
    }
 });
         setLocationRelativeTo(null);
         try {
              setLocationByPlatform(true);
         } catch(Throwable ignoreAndContinue) { }
          add(parametrPan,BorderLayout.WEST);
          pack();
          setVisible(true);
        } catch (ParseException ex) {
             JOptionPane.showMessageDialog(null,ex,"Error",JOptionPane.ERROR_MESSAGE);
        } finally{ if(con!=null)try {
               con.close();} catch (SQLException ex) {}
          }
  
   }

   
  void initData(){
      try{
           info.append("This is a info from resultset");
           add(info);
                 stmt  = con.prepareStatement(sessions);//clientInfo);
                 stmt.setString(1,cuid);
                    
                     Timestamp tsFrom = Timestamp.valueOf(dateFrom.getText());
                     Timestamp tsTo = Timestamp.valueOf(dateTo.getText());
                     System.out.println(tsFrom);
                     System.out.println(tsTo);
                     stmt.setString(2,tsFrom.toString());
                     stmt.setString(3,tsTo.toString());
                    // rs  = null;
                     rs  = stmt.executeQuery();
           Vector header = new Vector();
           header.addElement("Description");
           header.addElement("OpenTime");
           header.addElement("LastAccessTime");
           header.addElement("SessionID");
           header.addElement("Canal");
           header.addElement("TimeZone");
           header.addElement("View Logs");     
           Vector data = new Vector();
           int i=0;
           while(rs.next()) {
               Vector newRow = new Vector();
               newRow.addElement(rs.getString(1));
               newRow.addElement(rs.getTimestamp(2));
               newRow.addElement(rs.getTimestamp (3));
               newRow.addElement(rs.getString (4));
               newRow.addElement(rs.getString (5));
               newRow.addElement(rs.getString (6));
               newRow.addElement(" ");
               data.addElement(newRow);
               i++;
            }
     
           model = new DefaultTableModel(data, header);
           table = new JTable(model){
            public boolean isCellEditable(int row,int column){  
                    if (column==6)   return true;
                    else return false;  
            }
      
           };
           table.getTableHeader().setReorderingAllowed(false);
           logsCombo = new JComboBox();
           for(int j=0;j<3;j++){
               logsCombo.addItem(logsString[j]);
           }    
           
           table.getColumnModel().getColumn(6).setCellEditor(new DefaultCellEditor(logsCombo));
           
           tableScroll = new JScrollPane(table);
           add(tableScroll, BorderLayout.SOUTH);
           
           pack();
                } catch (SQLException ex) {
                  JOptionPane.showMessageDialog(null,ex,
                          "SQL Error",JOptionPane.ERROR_MESSAGE);   
                }
 }
     static String sessions="SELECT act.DESCRIPTION," +
"         SES.OPENTIME," +
"         SES.LASTACCESSTIME," +
"         SES.EXTERNALSESSIONID," +
"         SES.CHANNEL," +
"         SES.TIMEZONE" +          
"    FROM HC_GFO.GFO_PARTYCONTRACT pc2" +
"         JOIN HC_GFO.GFO_PARTYCONTRACT pc" +
"            ON (pc2.PARTY = pc.PARTY)" +
"         JOIN HC_GFO.GFO_ACTOR act" +
"            ON (pc2.ACTOR = act.ID)" +
"         JOIN HC_GFO.GFO_CLIENT cli" +
"            ON (pc2.CONTRACT = cli.CONTRACT)" +
"         JOIN HC_GFO.GFO_ACTOR act_" +
"            ON (pc.ACTOR = act_.ID)" +
"         JOIN HC_GFO.GFO_SESSIONACTORS SA" +
"            ON (act_.ID = SA.ACTORID)" +
"         JOIN HC_GFO.GFO_SESSION SES" +
"            ON (SA.SESSIONID = SES.ID)" +
"   WHERE (cli.EXTERNALID IN(?)) AND" +
"   (SES.OPENTIME BETWEEN to_timestamp(?, 'RRRR-MM-DD HH24:MI:SS.FF')"+
     "AND to_timestamp(?, 'RRRR-MM-DD HH24:MI:SS.FF'))" +
"   ORDER BY SES.OPENTIME ASC"; 

}
    /*
                     stmt  = con.prepareStatement(clientInfo);
                     stmt.setString(1,cuid);
                     rs  = stmt.executeQuery();
                   
                   while(rs.next()) {
                    //   for(int i=1; i<8;i++){
                          info.append(rs.getString(1)+"  ");
                          info.append(rs.getTimestamp(2).toString()+"  ");
                          info.append(rs.getTimestamp(3).toString()+"  ");
                          info.append(rs.getString(4)+"  ");
                          info.append(rs.getString(5)+"  ");
                          info.append("\n");
                      // }
                     }
                
                
       
                } 
        }
          });
         

 
//---------------------------------------------jsbc reqts---------------------------------------------------------------------  
   static String clientInfo= "select pc.*, pc2.*, act.*, act_.*," +
"  cr.*, cr_up.* , cnt_cr.* ,cli.*" +
"  from HC_GFO.gfo_partycontract pc" +
"  join HC_GFO.gfo_partycontract pc2 on pc2.party = pc.party and pc2.basedon is NULL" +
"  join HC_GFO.gfo_actor act on pc2.actor = act.id" +
"  join HC_GFO.gfo_actor act_ on pc.actor = act_.id" +
"  join HC_GFO.gfo_credentials cr on act_.id=cr.actor_id" +
"  join HC_GFO.gfo_up_credentials cr_up on cr.id = cr_up.id" +
"  join HC_GFO.gfo_client cli on pc2.contract = cli.contract" +
"  join HC_GFO.gfo_countable_credentials cnt_cr on cr.id = cnt_cr.id" +
"  where cli.externalid in ('32505782')";
    
  static String phoneByCuid="Select DISTINCT perc.value \n" +
"                   from hc_gfo.gfo_partycontract pc\n" +
"                   join hc_gfo.gfo_personcontact perc on perc.contacts_id = pc.party\n" +
"                   join hc_gfo.gfo_client cl on cl.contract = pc.basedon\n" +
"                   where cl.externalid in ('?') and perc.type = 'PO'"; 
   
 static  String trans="select \n" +
"   t.*, ft.initamount,\n" +
"   cli.externalid, act.description, accd.accountnumber as debitaccount, \n" +
"   (select accountnumber from HC_GFO.gfo_accountidentdata where id = ptr.creditaccountdata) as creditaccount,\n" +
"   (select BIC from HC_GFO.gfo_bankcode where code = (select bank from HC_GFO.gfo_accountidentdata where id = ptr.creditaccountdata)) as BIC,\n" +
"   (select value from HC_GFO.GFO_TRANSACTATTRIBUTE where type = 41 and transaction_id = t.id) as DEPOSIT_OPER,\n" +
"   (select max (statusdate) from HC_GFO.gfo_transactstatushistory tsh where tsh.status = t.status and tsh.transaction_id = t.id) as last_update,\n" +
"   ptr.*\n" +
"   from HC_GFO.gfo_transaction t\n" +
"   join HC_GFO.gfo_financialtransaction ft on t.id = ft.id\n" +
"   join HC_GFO.gfo_partycontract pc on pc.actor = t.registeredBy\n" +
"   join HC_GFO.gfo_partycontract pc2 on pc2.party = pc.party and pc2.basedon is NULL\n" +
"   join HC_GFO.gfo_actor act on pc2.actor = act.id\n" +
"   join HC_GFO.gfo_client cli on pc2.contract = cli.contract\n" +
"   join HC_GFO.gfo_paymenttransaction ptr on ptr.id = t.id\n" +
"   join HC_GFO.gfo_accountidentdata accd on accd.id = ptr.debitaccountdata\n" +
"   --join GFO_TRANSACTATTRIBUTE tr_attr on t.id = tr_attr.transaction_id\n" +
"   --join gfo_deposit_open_transaction dep_op on t.id = dep_op.id\n" +
"   where (cli.externalid in ('${cuid}'))AND\n" +
"   (t.registrationdate BETWEEN to_date('${dateFrom}','yyyy/mm/dd hh24:mi') AND to_date('${dateTo}','yyyy/mm/dd hh24:mi'))\n" +
"   order by 1";
   

}

*/

