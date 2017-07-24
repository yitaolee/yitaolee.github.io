---
layout: post
title: Encapsulate tableView, typealias
date: 2017-07-22 15:32:24.000000000 +09:00
---

#### What's the problem?

There are a lot of tableview on the app, there are a lot of common protocols or functions should be reused. 
Let the controllers do the control things.

Cell interactions are complicated, we better avoid a lot of code messing up with each parts.

####  

We use OO for this, because its enough for linear inherent, protocols are not necessary.  


````

import UIKit

class DVBaseVC: UIViewController , UIGestureRecognizerDelegate{
    
    var tapGesture:UITapGestureRecognizer?
    var autoHideKeyboard :Bool {
        set{
            if newValue == true {
                if tapGesture == nil {
                    tapGesture = UITapGestureRecognizer(target: self, action: #selector(self.onBackgroundTap))
                    tapGesture?.cancelsTouchesInView = false
                    tapGesture?.delegate = self
                    self.view.addGestureRecognizer(tapGesture!)
                }
            }else {
                if tapGesture != nil {
                    self.view.removeGestureRecognizer(tapGesture!)
                    tapGesture = nil
                }
            }
        }
        
        get {
            return (tapGesture != nil)
        }
        
    }
    
    func onBackgroundTap() {
        self.view.endEditing(true)
    }
    
    public func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldReceive touch: UITouch) -> Bool {
        if gestureRecognizer.isKind(of: UITapGestureRecognizer.classForCoder()) {
            if (touch.view?.isKind(of: UIControl.classForCoder()))! && !(touch.view?.isKind(of: UIScrollView.classForCoder()))! {
                return false
            }
        }
        return true
    }
    
    

    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.view.backgroundColor = UIColor.white;
//        self.edgesForExtendedLayout = []
        
        self.edgesForExtendedLayout = UIRectEdge()
    }
}


````

And

````

import UIKit
import SwiftyJSON
import ObjectMapper
import CoreLocation

class DVTbVC: DVBaseVC,UITableViewDelegate,UITableViewDataSource {
    var tableview:UITableView? // UI
    //var items:Array<DVTableItem>? //
    var sections:Array<Array<DVTableItem>>? //
    
    var enableHeaderRefresh = false
    var enableFooterRefresh = false
    
    private var rc:UIRefreshControl?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableview = UITableView(frame: CGRect(x: 0, y: 0, w: screen_width(), h: screen_height()-64), style: .plain)
        tableview?.backgroundView = nil
        tableview?.backgroundColor = UIColor.clear
        tableview?.delegate = self
        tableview?.dataSource = self
        
        tableview?.separatorColor = UIColor.clear
        tableview?.separatorStyle = .none
        tableview?.showsVerticalScrollIndicator = false
        self.view.addSubview(tableview!)
        
        self.resetRefreshControl()
        

    }
    
    func reloadByItems(list:Array<DVTableItem>?) {
        if list == nil {
            sections = [[]]
        }else {
            sections = [list!]
        }
        tableview?.reloadData()
    }

    func reloadByItemsBySection(list:Array<Array<DVTableItem>>?) {
        if list == nil {
            sections = [[]]
        }else {
            sections = list!
        }
        tableview?.reloadData()
    }

    
    public func numberOfSections(in tableView: UITableView) -> Int {
        if sections != nil {
            return (sections?.count)!
        }
        return 1
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        if sections != nil && section < (sections?.count)!{
            return (sections?[section].count)!
        }
        return 0
    }
    
    func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
        if sections != nil && (sections?.count)! > 1 {
            if(section == 0){
                return "Trending"
            }else{
                return "New created"
            }
        }else{
            return ""
        }
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        if sections == nil { //应该不会发生
            return UITableViewCell()
        }
        
        if indexPath.section >= (sections?.count)! {//应该不会发生
            return UITableViewCell()
        }
        
        let item:DVTableItem = (sections?[indexPath.section][indexPath.row])!
        
        var cell:DVTableCell?
        if (item.cellClassName != nil){
            var bcell = tableview?.dequeueReusableCell(withIdentifier: item.cellClassName!)
            if bcell == nil {
                //http://stackoverflow.com/questions/24030814/swift-language-nsclassfromstring

                if let implementationClass: NSObject.Type = NSClassFromString(item.cellClassName!) as? NSObject.Type{
                    bcell = implementationClass.init() as? DVTableCell
                }
                
            }
            cell = bcell as? DVTableCell
        }else if(item.cellNibName != nil){

            let bIden = item.cellNibName!
            var bcell = tableview?.dequeueReusableCell(withIdentifier: bIden)
            if bcell == nil {
                let nib = UINib(nibName: bIden, bundle: nil)
                tableview?.register(nib, forCellReuseIdentifier: bIden)
                bcell = tableview?.dequeueReusableCell(withIdentifier: bIden)
            }
            cell = bcell as? DVTableCell
        }
        
        if cell == nil {
            return UITableViewCell()
        }
        
        assert((cell?.isKind(of: DVTableCell.classForCoder()))!, "crash! cell 必须是 DVTableCell的子类")
        cell?.updateItem(aitem: item)
        
        return cell!
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        if sections == nil {
            return 0
        }
        
        if indexPath.section >= (sections?.count)! {
            return 0
        }
        
        let item:DVTableItem = (sections?[indexPath.section][indexPath.row])!
        return item.cellHeight
    }
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        if sections == nil {
            return
        }
        
        if indexPath.section >= (sections?.count)! {
            return
        }
        
        let item:DVTableItem = (sections?[indexPath.section][indexPath.row])!
        let cell:DVTableCell = tableview?.cellForRow(at: indexPath) as! DVTableCell
        if item.tapBlock != nil {
            item.tapBlock!(tableview!,indexPath,item,cell)
        }
        
    }
    
    

    func onFootLoadMore() {//next page data

    }
    
    func onHeadRefresh() {//refresh
        
    }
    
    func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        if indexPath.section >= (sections?.count)! {
            return
        }
        
        if indexPath.row == ((sections?[indexPath.section].count)! - 1) {//last row
            if enableFooterRefresh {
                self.onFootLoadMore()
            }
        }
    }
    
    func resetRefreshControl() {
        if enableHeaderRefresh == true {
            if rc == nil {
                rc = UIRefreshControl()
                rc?.addTarget(self, action: #selector(self.onHeadRefresh), for: .valueChanged)
                tableview!.addSubview(rc!)
            }
        }else{
            if rc != nil {
                rc?.removeFromSuperview()
                rc = nil
            }
        }
    }
    
    func endRefresh() {
        if rc != nil {
            rc?.endRefreshing()
        }
    }
    
    
}


````

For TableCell, typealias provides a way to redefine a make sense name for existing types.

````


import UIKit

typealias DVTbviewTapBlock = (_ tbview:UITableView , _ indexPath:IndexPath, _ item:DVTableItem, _ cell:DVTableCell) -> Void
typealias DVTbviewExtendBlock = (_ item:DVTableItem, _ cell:DVTableCell) -> Void
typealias DVTbviewSimpleTapBlock = (_ item:DVTableItem) -> Void

class DVTableItem: NSObject {
    var cellClassName:String?
    var cellNibName:String?
    var cellHeight : CGFloat = 44 
    
    var cellData:AnyObject? //Data
    
    //
    var tapBlock:DVTbviewTapBlock?
    var tapExtendBlk:DVTbviewExtendBlock?
    
    var cacheDict:NSMutableDictionary = NSMutableDictionary() //Provide a way send the data in.
    

}

@objc(DVTableCell)
class DVTableCell: UITableViewCell {
    
    var arrowView:UIImageView?
    var bottomLine:UIImageView?
    
    var item:DVTableItem?
    
    var hasSetupUI = false
    
    var showBottomLine:Bool {
        set{
            if newValue == true {
                self.contentView.bringSubview(toFront: self.bottomLine!)
                bottomLine?.isHidden = false
            }else{
                bottomLine?.isHidden = true
            }
        }
        get {
            return false
        }
    }
    
    var showRightArrow:Bool {
        set{
            if newValue == true {
                self.contentView.bringSubview(toFront: self.arrowView!)
                arrowView?.isHidden = false
            }else{
                arrowView?.isHidden = true
            }
        }
        get {
            return false
        }
    }
    
    //MARK: -
    
    init() {
        let iden = typeName(some: DVTableCell.self)
        super.init(style: .default, reuseIdentifier: iden)
        
        self.setupTableCell()
    }
    
    
    override init(style: UITableViewCellStyle, reuseIdentifier: String!) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        self.setupTableCell()
    }
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        //fatalError("init(coder:) has not been implemented")
    }
    
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        self.setupTableCell()
        
    }
    
    func updateItem(aitem:DVTableItem?) {
        self.item = aitem
    }
    
    func setupTableCell() {
        self.selectionStyle = .none
        self.backgroundColor = UIColor.clear
        
        bottomLine = makeSepline()
        self.contentView.addSubview(bottomLine!)
        
        let img = UIImage(named: "icon_arrow_right")
        arrowView = UIImageView(image: img)
        self.contentView .addSubview(arrowView!)
        
        bottomLine?.snp.makeConstraints({ (make) in
            make.width.equalTo(self.snp.width);
            make.height.equalTo(0.5);
            make.centerX.equalTo(self);
            make.bottom.equalTo(self.snp.bottom);
        })
        
        arrowView?.snp.makeConstraints({ (make) in
            make.size.equalTo(arrowView!)
            make.centerX.equalTo(self);
            make.right.equalTo(self.snp.right).inset(15)
        })
        
        self.showRightArrow = false
        self.showBottomLine = false
    }
    
    
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
        
        // Configure the view for the selected state
    }
    
}

func makeCodeTbItem(cellClassName:String? ,data:AnyObject? ,tapblk:DVTbviewTapBlock?) -> DVTableItem{
    let item = DVTableItem()
    item.cellClassName = cellClassName
    item.cellData = data
    item.tapBlock = tapblk
    return item
}

func makeNibTbItem(cellNibName:String? ,data:AnyObject? ,tapblk:DVTbviewTapBlock?) -> DVTableItem{
    let item = DVTableItem()
    item.cellNibName = cellNibName
    item.cellData = data
    item.tapBlock = tapblk
    return item
}



````
















