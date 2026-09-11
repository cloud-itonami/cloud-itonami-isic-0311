(ns fishing.sim
  "Simulation/demo: drive the marine fishing fleet operations actor through
  a few scenarios."
  (:require [fishing.store :as store]
            [fishing.operation :as operation]))

(defn -main []
  (println "Marine Fishing Fleet Operations Coordination Actor - Demo")
  (println "===========================================================")

  ;; Setup: create store with a test vessel.
  (let [test-vessel {:vessel-id "fishing-vessel-001"
                     :name "Ocean Hope"
                     :status :active
                     :flag-state "JP"
                     :permit-number "PERMIT-001-2026"
                     :quota-kg 50000
                     :landed-kg 0}
        st (store/mem-store {:fixtures {"fishing-vessel-001" test-vessel}})

        ;; Build the operation actor.
        actor (operation/build st)]

    (println "\nVessel registered:")
    (println "  Vessel ID:" (:vessel-id test-vessel))
    (println "  Name:" (:name test-vessel))
    (println "  Flag state:" (:flag-state test-vessel))
    (println "  Permit:" (:permit-number test-vessel))
    (println "  Quota (kg):" (:quota-kg test-vessel))

    (println "\nOperation Actor built and ready.")
    (println "Governance rules active:")
    (println "  - Vessel/permit verification required")
    (println "  - All effects :propose only")
    (println "  - Safety concerns always escalate immediately")
    (println "  - High-cost supply orders escalate")
    (println "  - Quota-exceedance catch records are hard-blocked (permanent)")
    (println "  - Navigation / vessel-command / catch decisions are blocked (captain's exclusive authority)")))
