---
layout: post
title:  "iOS Workshop"
permalink: /iosworkshop
date:   2019-10-15 18:51:13 -0500
categories: ios
---
{% highlight swift %}
//
//  helpers.swift
//  AllowanceCoreData
//
import Foundation

func formatCurrency(amount : Int) -> String
{
    let amt = Double(amount)
    if(amount > 0){
        return String(format:"$%.2f",amt/100)
    }
    else{
        return String(format:"($%.2f)",amt/100)
    }
}
{% endhighlight %}




{% highlight swift %}
//
//  ViewController.swift
//  AllowanceCoreData
//

import UIKit
import CoreData

var transactions: [NSManagedObject] = []


class TransactionTableViewController: UITableViewController {


    override func viewDidLoad() {
        super.viewDidLoad()
        NotificationCenter.default.addObserver(self,
                                               selector: #selector(self.handleModalDismissed),
                                               name: NSNotification.Name(rawValue: "modalIsDimissed"),
                                               object: nil)    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        //1
        guard let appDelegate =
            UIApplication.shared.delegate as? AppDelegate else {
                return
        }

        let managedContext =
            appDelegate.persistentContainer.viewContext

        //2
        let fetchRequest =
            NSFetchRequest<NSManagedObject>(entityName: "Transaction")

        //3
        do {
            transactions = try managedContext.fetch(fetchRequest)
        } catch let error as NSError {
            print("Could not fetch. \(error), \(error.userInfo)")
        }
    }

    override func tableView(_ tableView: UITableView,
                            numberOfRowsInSection section: Int) -> Int {
        return transactions.count
    }

    override func tableView(_ tableView: UITableView,
                            cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {

            let transaction = transactions[indexPath.row]
            let cell =
                tableView.dequeueReusableCell(withIdentifier: "cell_transaction",
                                              for: indexPath)
            cell.textLabel?.text =
                transaction.value(forKeyPath: "payee") as? String
            let amt = transaction.value(forKeyPath: "amount") as! Int
            cell.detailTextLabel?.text = formatCurrency(amount: amt)
            return cell
    }

    //delete
    override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath)
    {

        if indexPath.row < transactions.count
        {

            //1
            guard let appDelegate =
                UIApplication.shared.delegate as? AppDelegate else {
                    return
            }

            let managedContext =
                appDelegate.persistentContainer.viewContext

            let transaction = transactions[indexPath.row]
            let deleteRequest = NSBatchDeleteRequest(objectIDs: [transaction.objectID])

            do {
                try managedContext.execute(deleteRequest)
            } catch let error as NSError {
                print(error)
            }
            transactions.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .top)
        }
    }

    @objc func handleModalDismissed() {
        tableView.reloadData()
    }



}



class AddTransactionViewController: UIViewController {
    @IBOutlet weak var payeeTF: UITextField!
    @IBOutlet weak var amtTF: UITextField!

    @IBOutlet weak var withdrawDepositSC: UISegmentedControl!

    override func viewDidLoad() {
        super.viewDidLoad()

    }

    @IBAction func addTransaction(_ sender: Any) {


        guard let payee = payeeTF.text else{
            return
        }
        guard let amount = amtTF.text else{
            return
        }
        var  amt = Int(100*(Double(amount) ?? 0))
        if withdrawDepositSC.selectedSegmentIndex == 0{
           amt *= -1
        }


        self.save(payee: payee, amount: amt)
        NotificationCenter.default.post(name: NSNotification.Name(rawValue: "modalIsDimissed"), object: nil)
        self.dismiss(animated: true, completion: nil)

    }


    func save(payee: String, amount: Int) {

        guard let appDelegate =
            UIApplication.shared.delegate as? AppDelegate else {
                return
        }

        // 1
        let managedContext =
            appDelegate.persistentContainer.viewContext

        // 2
        let entity =
            NSEntityDescription.entity(forEntityName: "Transaction",
                                       in: managedContext)!

        let transaction = NSManagedObject(entity: entity,
                                          insertInto: managedContext)

        // 3
        transaction.setValue(payee, forKeyPath: "payee")
        transaction.setValue(amount, forKeyPath: "amount")

        // 4
        do {
            try managedContext.save()
            transactions.append(transaction)
        } catch let error as NSError {
            print("Could not save. \(error), \(error.userInfo)")
        }
    }



}

class StatisticsVC : UIViewController{

    @IBOutlet weak var balanceLabel: UILabel!
    override func viewDidLoad() {
        updateBalance()
    }
    override func viewWillAppear(_ animated: Bool) {
        updateBalance()
    }



    func updateBalance(){
        var balance = 0
        for t in transactions{
            let amt = t.value(forKeyPath: "amount") as! Int
            balance += amt
        }
        balanceLabel.text = formatCurrency(amount: balance)
    }

    @IBAction func deleteAll(_ sender: Any) {
        //1
        guard let appDelegate =
            UIApplication.shared.delegate as? AppDelegate else {
                return
        }

        let managedContext =
            appDelegate.persistentContainer.viewContext

        for t in transactions{

        let deleteRequest = NSBatchDeleteRequest(objectIDs: [t.objectID])

        do {
            try managedContext.execute(deleteRequest)
        } catch let error as NSError {
            print(error)
        }
        }

    }
}

{% endhighlight %}
